# Example for DTO without AutoMapper

1. Create a new Web API Project (CF_DTO_without_AutoMapper_Demo)
2. Add Models Folder ==> Add Book Class
 ```cs
 public class Book
 {
     public int Id { get; set; }
     public string Title { get; set; }
     public string Author { get; set; }
     public decimal Price { get; set; }
     public DateTime? ReleaseDate { get; set; }
     public string? Genre { get; set; }
 }
    ```
4. Add DTO Folder  - Add BookDTO Class
```cs
    public class BookDTO
    {
        public int Id { get; set; }
        public string Title { get; set; }
        public string Author { get; set; }
        public decimal Price { get; set; }
    }
```

install below packages using Nuget package Manager solution
```cs 
Microsoft.AspNetCore.JsonPatch
Microsoft.AspNetCore.Mvc.NewtonsoftJson
Microsoft.EntityFrameworkCore
Microsoft.EntityFrameworkCore.SqlServer
Microsoft.EntityFrameworkCore.Tools
```
5. Add Data Folder -Add ApplicationDbContext class
```cs
    public class ApplicationDbContext:DbContext
 {
     public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options)
     {
     }

     public DbSet<Book> Books { get; set; }
 }
```
# appsetting.json
```json
    ,
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\MSSQLLocalDB;Database=Book_DB;Trusted_Connection=True;TrustServerCertificate=true;"
  }

```
# Program.cs
```cs
    builder.Services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

          builder.Services.AddControllers().AddNewtonsoftJson();
```
Create a folder Name it as Utility (or) Mappings
Add class MappingExtensions.cs
```cs
using CF_DTO_without_AutoMapper_Demo.DTO;
using CF_DTO_without_AutoMapper_Demo.Models;

namespace CF_DTO_without_AutoMapper_Demo.Utility
{
    public static class MappingExtensions
    {
        public static BookDTO ToDTO(this Book book)
        {
            return new BookDTO
            {
                Id = book.Id,
                Title = book.Title,
                Author = book.Author,
                Price = book.Price,
            };
        }

        public static Book ToEntity(this BookDTO bookDTO)
        {
            return new Book
            {
                Id = bookDTO.Id,
                Title = bookDTO.Title,
                Author = bookDTO.Author,
                Price = bookDTO.Price,
            };
        }
        public static List<BookDTO> ToDTOList(this List<Book> books)
        {
            var bookDTOs = new List<BookDTO>();

            //foreach (var book in books)
            //{
            //    bookDTOs.Add(book.ToDTO());
            //}
           // return bookDTOs;
            //or
            return books.Select(book => book.ToDTO()).ToList();
           
        }
    }
}

```

Create a controller ==> BooksController

```cs
using CF_DTO_without_AutoMapper_Demo.Data;
using CF_DTO_without_AutoMapper_Demo.DTO;
using CF_DTO_without_AutoMapper_Demo.Models;
using CF_DTO_without_AutoMapper_Demo.Utility;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.JsonPatch;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using Microsoft.AspNetCore.Mvc.NewtonsoftJson;

namespace CF_DTO_without_AutoMapper_Demo.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class BooksController : ControllerBase
    {
        
    private readonly ApplicationDbContext _context;

        public BooksController(ApplicationDbContext context)
        {
            _context = context;
        }
        [HttpGet]
        public async Task<ActionResult<List<BookDTO>>> GetBooks()
        {
            List<Book> books= await _context.Books.ToListAsync();
            if (books != null)
            {
                List<BookDTO> booksDTO = books.ToDTOList();
                return booksDTO;                
            }
            else
            {
                return NotFound();
            }
        }

        [HttpGet("{id}")]
        public async Task<ActionResult<BookDTO>> GetBook(int id)
        {
            var book = await _context.Books.FindAsync(id);
            if (book == null)
            {
                return NotFound();
            }

            var bookDTO = book.ToDTO();
            return bookDTO;
        }

        [HttpPost]
        public async Task<ActionResult<BookDTO>> PostBook(BookDTO bookDTO)
        {
            var book = bookDTO.ToEntity();
            _context.Books.Add(book);
            await _context.SaveChangesAsync();

            return CreatedAtAction(nameof(GetBook), new { id = book.Id }, bookDTO);
        }

        [HttpPut("{id}")]
        public IActionResult UpdateBook(int id, [FromBody] BookDTO bookDto)
        {
            var book = _context.Books.FirstOrDefault(b => b.Id == id);
            if (book == null)
            {
                return NotFound();
            }

            book.Title = bookDto.Title;
            book.Author = bookDto.Author;
            book.Price = bookDto.Price;

            _context.SaveChanges();

            return NoContent();
        }

        [HttpPatch("{id}")]
        public IActionResult PartiallyUpdateBook(int id, [FromBody] JsonPatchDocument<BookDTO> patchDoc)
        {
            var book = _context.Books.FirstOrDefault(b => b.Id == id);
            if (book == null)
            {
                return NotFound();
            }

            var bookDto = new BookDTO
            {
                Id = book.Id,
                Title = book.Title,
                Author = book.Author,
                Price = book.Price
            };
           
            patchDoc.ApplyTo(bookDto,ModelState);

            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState);
            }

            book.Title = bookDto.Title;
            book.Author = bookDto.Author;
            book.Price = bookDto.Price;
           _context.SaveChanges();
            return NoContent();
        }
        [HttpDelete("{id}")]
        public IActionResult DeleteBook(int id)
        {
            var book = _context.Books.FirstOrDefault(b => b.Id == id);
            if (book == null)
            {
                return NotFound();
            }

            _context.Books.Remove(book);
            _context.SaveChanges();
            return NoContent();
        }

        }
    }


```
# Explanation
**Entity (Book):** Represents the database structure.

**DTO (BookDTO):** Simplified version for data transfer.

**DbContext (ApplicationDbContext):** Manages your database connection and transactions.

# Steps to perform Patch in swagger
![alt text](image.png)
[
  {
    "op": "replace",
    "path": "/title",
    "value": "Updated Title"
  },
  {
    "op": "replace",
    "path": "/price",
    "value": 99.99
  }
]

# Alternate way using AutoMapper
create a new Web APIproject (**API_DTO_AutoMapper_Demo**)

add Models Folder ==> Add Book.cs

Step 2: Define Your Entity

Create a Book entity class.
```cs
public class Book
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Author { get; set; }
    public decimal Price { get; set; }
    public DateTime ReleaseDate { get; set; }
    public string Genre { get; set; }
}
```
**Step 3:** Define Your DTO
Create a BookDTO class.
```cs
public class BookDTO
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Author { get; set; }
    public decimal Price { get; set; }
}
```

# install below packages from Nuget
```
<PackageReference Include="AutoMapper" Version="13.0.1" />
  
<PackageReference Include="Microsoft.AspNetCore.JsonPatch" Version="8.0.8" />
<PackageReference Include="Microsoft.AspNetCore.Mvc.NewtonsoftJson" Version="8.0.8" />
<PackageReference Include="Swashbuckle.AspNetCore" Version="6.4.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore" Version="8.0.8" />
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.8" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="8.0.8" />
```

# appsetting.json
```json
 "ConnectionStrings": {
   "DefaultConnection": "Server=(localdb)\\MSSQLLocalDB;Database=Book_DB;Trusted_Connection=True;TrustServerCertificate=true;"
 }
```
**Step 4:** Set Up Your DbContext
Create ApplicationDbContext to manage your database.
```cs
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options)
    {
    }

    public DbSet<Book> Books { get; set; }
}
```
**Step 5:** Configure Your DbContext
 # Program.cs 
```cs
builder.Services.AddAutoMapper(typeof(BookProfile));
builder.Services.AddLogging();
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
builder.Services.AddControllers().AddNewtonsoftJson();
```
**Step 6:** Create AutoMapper Profile
create a Folder with the name of Profiles

Create a mapping profile for AutoMapper. 
**class with the name of MappingProfile **
```cs
public class MappingProfile : Profile
{
    public MappingProfile()
    {
        CreateMap<Book, BookDTO>();
        CreateMap<BookDTO, Book>();

       // (or)
        CreateMap<Book,BookDTO>().ReverseMap();
    }
}
```
**Step 7:** Update Your Controller
In your controller, inject the ApplicationDbContext and IMapper.

```csharp
using API_DTO_AutoMapper_Demo.DTO;
using API_DTO_AutoMapper_Demo.Models;
using AutoMapper;
using CF_DTO_without_AutoMapper_Demo.Data;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.JsonPatch;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Logging;

namespace API_DTO_AutoMapper_Demo.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class BooksController : ControllerBase
    {
        private readonly IMapper _mapper;
        private readonly ApplicationDbContext _context;
        private readonly ILogger<BooksController> _logger;
        public BooksController(IMapper mapper,ApplicationDbContext context,ILogger<BooksController>  logger)
        {
            _mapper = mapper;
            _context = context;
            _logger = logger;
        }
        
        [HttpPatch("{id}")]
        public IActionResult PartiallyUpdateBook(int id, [FromBody] JsonPatchDocument<BookDTO> patchDoc)
        {
            var book = _context.Books.FirstOrDefault(b => b.Id == id);
            if (book == null)
            {
                return NotFound();
            }

            //// Create BookDTO from Book
            var bookDto = new BookDTO
            {
               Id = book.Id,
               Title = book.Title,
               Author = book.Author,
               Price = book.Price
            };
            
            // Apply the patch to BookDTO
            patchDoc.ApplyTo(bookDto, ModelState);
            _logger.LogInformation("Patched BookDTO: {@BookDTO}", bookDto);
            // Check for validation errors
            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState);
            }

            // Update Book with patched values
            book.Title = bookDto.Title;
            book.Author = bookDto.Author;
            book.Price = bookDto.Price;

            // Ensure the entity is tracked by the context
            _context.Entry(book).State = EntityState.Modified;

            // Save changes to the database
            _context.SaveChanges();

            return NoContent();
        }
       

    }
}

```


**AutoMapper:** Simplifies the mapping between entities and DTOs.

This setup ensures you’re efficiently handling data between your API 
and the database while maintaining a clean separation between the data 
and presentation layers. Keeps the codebase tidy and maintainable.

# The same will acheive using Record

instead of BookDTO class you can use Record in below format

![alt text](image-1.png)
```cs
namespace API_DTO_AutoMapper_Demo.DTO
{
   
    public record BookDTO(int Id, string Title, string Author, decimal Price);

}

```


# BooksController.cs
**HttpGET,HttpGetbyid,Post,put and delete all methods have same code as above code in books controller.cs**
```cs
using API_DTO_AutoMapper_Demo.DTO;
using API_DTO_AutoMapper_Demo.Models;
using AutoMapper;
using CF_DTO_without_AutoMapper_Demo.Data;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.JsonPatch;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Logging;

namespace API_DTO_AutoMapper_Demo.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class BooksController : ControllerBase
    {
        private readonly IMapper _mapper;
        private readonly ApplicationDbContext _context;
        private readonly ILogger<BooksController> _logger;
        public BooksController(IMapper mapper,ApplicationDbContext context,ILogger<BooksController>  logger)
        {
            _mapper = mapper;
            _context = context;
            _logger = logger;
        }
        
        
        [HttpPatch("{id}")]
        public IActionResult PartiallyUpdateBook(int id, [FromBody] JsonPatchDocument<BookDTO> patchDoc)
        {
            var book = _context.Books.FirstOrDefault(b => b.Id == id);
            if (book == null)
            {
                return NotFound();
            }

            ////// Create BookDTO from Book
           
            // Create BookDTO from Book
            var bookDto = new BookDTO(book.Id, book.Title, book.Author, book.Price);
            // Apply the patch to BookDTO
            patchDoc.ApplyTo(bookDto, ModelState);
            _logger.LogInformation("Patched BookDTO: {@BookDTO}", bookDto);
            // Check for validation errors
            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState);
            }

            // Update Book with patched values
            book.Title = bookDto.Title;
            book.Author = bookDto.Author;
            book.Price = bookDto.Price;

            // Ensure the entity is tracked by the context
            _context.Entry(book).State = EntityState.Modified;

            // Save changes to the database
            _context.SaveChanges();

            return NoContent();
        }      

    }
}
```