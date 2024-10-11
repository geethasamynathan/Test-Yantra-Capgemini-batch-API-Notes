# step 1
create new MVC Project (Consume_API_MVC)

from the continuation of completing webAPI
https://github.com/geethasamynathan/JWT_Auth_Demo_TestYantra

# appsetting.json
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ApiSettings": {    
    "AuthURL": "https://localhost:7093/api/Authenticate/"
  }
}
```
# program.cs
```cs
builder.Services.AddHttpClient();
```

# LoginModel.cs in Models Folder 
```cs
using System.ComponentModel.DataAnnotations;

namespace Trg_Consume_API_in_MVC.Models
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

create a new Controller
# LoginController.cs
```cs
using JWT_Auth_Demo.Authentication;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Mvc;
using Newtonsoft.Json;
using NuGet.Configuration;
using NuGet.Protocol;
using System.Text;

namespace Consume_API_MVC.Controllers
{
    public class LoginController : Controller
    {
        private readonly HttpClient _httpClient;
        private readonly IConfiguration _config;
        public LoginController(HttpClient httpClient,
            IConfiguration configuration)
        {
            _httpClient = httpClient;
            _config = configuration;

        }
        public IActionResult Index()
        {
            var token = TempData["Token"] as string;
            ViewBag.Token = token;
            return View();
        }
        [HttpGet]
        public IActionResult Login()
        {
            return View();
        }
        [HttpPost]
        public async Task<IActionResult> Login(LoginModel login)
        {
            if (ModelState.IsValid)
            {
                var json = JsonConvert.SerializeObject(login);
                var content = new StringContent(json, Encoding.UTF8, "application/json");
                var response = await _httpClient.PostAsync($"{_config["ApiSettings:AuthURL"]}login", content);
                if (response.IsSuccessStatusCode)
                {
                    var responseString = await response.Content.ReadAsStringAsync();
                    var tokenResponse = JsonConvert.DeserializeObject<Dictionary<string, string>>(responseString);
                    var token = tokenResponse["token"];
                    TempData["Token"] = token;
                    // return RedirectToAction("Index");
                    TempData["SuccessMessage"] = "Login success!";
                    return RedirectToAction("Index");
                }
                else
                {
                    TempData["ErrorMessage"] = "Login Failed.";
                    ModelState.AddModelError("", "Login failed");
                    return View();
                }
            }
            else
            {
                {
                    TempData["ErrorMessage"] = "Login Failed.";
                   // ModelState.AddModelError("", "Login failed");
                    return View();

                }
            }
        }
    }
}
```

