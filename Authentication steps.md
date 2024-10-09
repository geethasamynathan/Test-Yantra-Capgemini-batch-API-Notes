# authentication in web api Demo
1. create a new project (Training_Auth_Demo)

2. # Packages needs tom install
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
            "Key": "MySuperSecretKey12345",
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
            ValidAudience = builder.Configuration["JWT:ValidAudience"],
            ValidIssuer = builder.Configuration["JWT:ValidIssuer"],
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