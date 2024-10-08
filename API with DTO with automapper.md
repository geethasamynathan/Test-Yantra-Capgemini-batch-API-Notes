# Example for DTO with AutoMapper

1. Create a new Web API Project
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
```
Create a folder Name it as Utility (or) Mappings
Add class MappingExtensions.cs
```cs
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




# Alternate way using AutoMapper

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
In Startup.cs or Program.cs (depending on .NET version), configure your DbContext.
```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));

    services.AddControllers();

    // Add AutoMapper
    services.AddAutoMapper(AppDomain.CurrentDomain.GetAssemblies());
}
```
**Step 6:** Create AutoMapper Profile
Create a mapping profile for AutoMapper.
```cs
public class MappingProfile : Profile
{
    public MappingProfile()
    {
        CreateMap<Book, BookDTO>();
        CreateMap<BookDTO, Book>();
    }
}
```
**Step 7:** Update Your Controller
In your controller, inject the ApplicationDbContext and IMapper.

```csharp
[ApiController]
[Route("api/[controller]")]
public class BooksController : ControllerBase
{
    private readonly ApplicationDbContext _context;
    private readonly IMapper _mapper;

    public BooksController(ApplicationDbContext context, IMapper mapper)
    {
        _context = context;
        _mapper = mapper;
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<BookDTO>> GetBook(int id)
    {
        var book = await _context.Books.FindAsync(id);
        if (book == null)
        {
            return NotFound();
        }

        // Use AutoMapper to map Book to BookDTO
        var bookDTO = _mapper.Map<BookDTO>(book);
        return bookDTO;
    }

    [HttpPost]
    public async Task<ActionResult<BookDTO>> PostBook(BookDTO bookDTO)
    {
        var book = _mapper.Map<Book>(bookDTO);
        _context.Books.Add(book);
        await _context.SaveChangesAsync();

        return CreatedAtAction(nameof(GetBook), new { id = book.Id }, bookDTO);
    }
}
```


**AutoMapper:** Simplifies the mapping between entities and DTOs.

This setup ensures you’re efficiently handling data between your API and the database while maintaining a clean separation between the data and presentation layers. Keeps the codebase tidy and maintainable.