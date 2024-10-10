# authentication in web api Demo
1. create a new project (Training_Auth_Demo)

2. # Packages needs to install
  <PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="8.0.10" />
  <PackageReference Include="Microsoft.AspNetCore.Identity.EntityFrameworkCore" Version="8.0.10" />
  <PackageReference Include="Microsoft.EntityFrameworkCore" Version="8.0.10" />
  <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.10" />
  <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="8.0.10">
3.create an Authentication Folder

    a) add new class (ApplicationUser)

 ```cs
    using Microsoft.AspNetCore.Identity;

    namespace Training_Auth_Demo.Authentication
    {
        public class ApplicationUser:IdentityUser
        {
        }
    }

 ```
    b) add another class in authentication Folder (LoginModel)
```cs
        using System.ComponentModel.DataAnnotations;

        namespace Training_Auth_Demo.Authentication
        {
            public class LoginModel
            {
                [Required(ErrorMessage = "User Name is required")]
                public string Username { get; set; }

                [Required(ErrorMessage = "Password is required")]
                public string Password { get; set; }
            }
        }

```
    c) Add another class in authentication folder (RegisterModel)
```cs
    using System.ComponentModel.DataAnnotations;

        namespace JWT_Auth_Demo.Authentication
        {
            public class RegisterModel
            {
                [Required(ErrorMessage = "User Name is required")]
                public string Username { get; set; }

                [EmailAddress]
                [Required(ErrorMessage = "Email is required")]
                public string Email { get; set; }

                [Required(ErrorMessage = "Password is required")]
                public string Password { get; set; }
            }
        }
```

    d) Add another class in authentication folder (UserRoles)
```cs
            public static class UserRoles
        {
            public const string Admin = "Admin";
            public const string User = "User";
        }
    ```

    e) Add another class in authentication folder (Response)
```cs
        namespace Training_Auth_Demo.Authentication
        {
            public class Response
            {
                public string Status { get; set; }
                public string Message { get; set; }
            }
        }

```

f) Add another class in authentication folder ApplicationDbContext
 ```cs
    using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
    using Microsoft.EntityFrameworkCore;

    namespace Training_Auth_Demo.Authentication
    {
        public class ApplicationDbContext:IdentityDbContext<ApplicationUser>
        {
            public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) :
                base(options) { }
        }
    }

 ```
# appsettings.json
```json
        {
        "Logging": {
            "LogLevel": {
            "Default": "Information",
            "Microsoft.AspNetCore": "Warning"
            }
        },
        "AllowedHosts": "*",
        "Jwt": {
            "Key": "MySuperSecretKey12345Algorithms",
            "Issuer": "yourdomain.com",
            "Audience": "yourdomain.com"
        },
        "ConnectionStrings": {
            "DefaultConnection": "Server=(localdb)\\MSSQLLocalDB;Database=trg_Auth_DB_TestYantra;Trusted_Connection=true;TrustServerCertificate=true;"
        }
}

```

# Program.cs
```cs
using JWT_Auth_Demo.Authentication;
using JWT_Auth_Demo.Models;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using Microsoft.IdentityModel.Tokens;
using System.Text;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
{
    builder.Services.AddDbContext<ApplicationDbContext>(options => options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")));
    // For Identity
  builder.Services.AddIdentity<ApplicationUser, IdentityRole>()
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultTokenProviders();

    // Adding Authentication
    builder.Services.AddAuthentication(options =>
    {
        options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
        options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
        options.DefaultScheme = JwtBearerDefaults.AuthenticationScheme;
    })

    // Adding Jwt Bearer
    .AddJwtBearer(options =>
    {
        options.SaveToken = true;
        options.RequireHttpsMetadata = false;
        options.TokenValidationParameters = new TokenValidationParameters()
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidAudience = builder.Configuration["JWT:Audience"],
            ValidIssuer = builder.Configuration["JWT:Issuer"],
            IssuerSigningKey = new SymmetricSecurityKey
            (Encoding.UTF8.GetBytes(builder.Configuration["JWT:Key"]))
        };
    });


builder.Services.AddControllers();
          // Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
          builder.Services.AddEndpointsApiExplorer();
          builder.Services.AddSwaggerGen();

          var app = builder.Build();

          // Configure the HTTP request pipeline.
          if (app.Environment.IsDevelopment())
          {
              app.UseSwagger();
              app.UseSwaggerUI();
          }

          app.UseHttpsRedirection();
          app.UseAuthentication();
          app.UseAuthorization();

          app.MapControllers();

          app.Run();
      }

      
```

Build your solution


Package manager console

Add-Migration "Db creation"

update-database

# Add AuthenticationController 
```cs
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;
using System.Net;
using System.Security.Claims;
using System.Text;
using Training_Auth_Demo.Authentication;

namespace Training_Auth_Demo.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class AuthenticationController : ControllerBase
    {
        private readonly UserManager<ApplicationUser> userManager;
        private readonly RoleManager<IdentityRole> roleManager;
        private readonly IConfiguration _configuration;
        public AuthenticationController(UserManager<ApplicationUser> userManager,
            RoleManager<IdentityRole> roleManager,IConfiguration configuration)
        {
            this.userManager = userManager;
            this.roleManager = roleManager;
            _configuration = configuration;
        }

        [HttpPost]
        [Route("login")]
        public async Task<IActionResult> Login([FromBody] LoginModel model)
        {
            var user= await userManager.FindByNameAsync(model.Username);
            if (user !=null && await userManager.CheckPasswordAsync(user,model.Password))
            {
            var userRoles=await userManager.GetRolesAsync(user);
                var authClaims = new List<Claim>
                {  new Claim(ClaimTypes.Name,user.UserName),
                new Claim(JwtRegisteredClaimNames.Jti,Guid.NewGuid().ToString())                
                };
                foreach (var userRole in userRoles)
                {
                    authClaims.Add(new Claim(ClaimTypes.Role, userRole));
                }

                var authSigninKey = new SymmetricSecurityKey(
                    Encoding.UTF8.GetBytes(_configuration["Jwt:Key"]));
                var token = new JwtSecurityToken(
                    issuer: _configuration["Jwt:Issuer"],
                    audience: _configuration["Jwt:Audience"],
                    expires: DateTime.Now.AddHours(5),
                    claims: authClaims,
                    signingCredentials: new SigningCredentials(authSigninKey,
                    SecurityAlgorithms.HmacSha256)
                    );

                return Ok(new
                {
                    token = new JwtSecurityTokenHandler().WriteToken(token),
                    expiration = token.ValidTo
                });
            }
            return Unauthorized();
        }
        [HttpPost]
        [Route("register")]
        public async Task<IActionResult> Register(RegisterModel model)
        {
            var userExists = await userManager.FindByNameAsync(model.Username);
                if(userExists != null)            
                return StatusCode(StatusCodes.Status500InternalServerError,
                    new Response { Status = "Error", Message = "User Exists Already! " });
            ApplicationUser user = new ApplicationUser()
            {
                Email = model.Email,
                SecurityStamp = Guid.NewGuid().ToString(),
                UserName = model.Username
            };
             var result=await userManager.CreateAsync(user);
            if(!result.Succeeded)
            {
                return StatusCode(StatusCodes.Status500InternalServerError,
                    new Response
                    {
                        Status = "Error",
                        Message = "User creation failed! Please check user details and try again. "
                    });
            } 
            if(await roleManager.RoleExistsAsync(UserRoles.User))
            {
                await userManager.AddToRoleAsync(user, UserRoles.User);
            }
            return Ok(new Response
            {
                Status = "Success",
                Message = "User Added Successfully"
            });
        }

        [HttpPost]
        [Route("register-admin")]
        public async Task<IActionResult> RegisterAdmin([FromBody] RegisterModel model)
        {
            var userExists = await userManager.FindByNameAsync(model.Username);
            if (userExists != null)
                return StatusCode(StatusCodes.Status500InternalServerError,
                    new Response { Status = "Error", Message = "User already exists!" });

            ApplicationUser user = new ApplicationUser()
            {
                Email = model.Email,
                SecurityStamp = Guid.NewGuid().ToString(),
                UserName = model.Username
            };
            var result = await userManager.CreateAsync(user, model.Password);
            if (!result.Succeeded)
                return StatusCode(StatusCodes.Status500InternalServerError,
                    new Response
                    {
                        Status = "Error",
                        Message = "User creation failed! Please check user details and try again."
                    });

            if (!await roleManager.RoleExistsAsync(UserRoles.Admin))
                await roleManager.CreateAsync(new IdentityRole(UserRoles.Admin));
            if (!await roleManager.RoleExistsAsync(UserRoles.User))
                await roleManager.CreateAsync(new IdentityRole(UserRoles.User));

            if (await roleManager.RoleExistsAsync(UserRoles.Admin))
            {
                await userManager.AddToRoleAsync(user, UserRoles.Admin);
            }

            return Ok(new Response { Status = "Success", Message = "User created successfully!" });
        }
    }
    }


```
