# What is CORS in asp.net core web API?
**Cross-Origin Resource Sharing (CORS)** is a security feature implemented by web browsers to allow or restrict web applications running on one domain to make requests to another domain. In ASP.NET Core Web API 8.0, CORS is used to relax the same-origin policy, which prevents a web page from making requests to a different domain than the one that served the web page.

**How CORS works in ASP.NET Core Web API 8.0:**

# Enabling CORS
There are several ways to enable CORS in ASP.NET Core Web API 8.0:

**Using Middleware:** You can configure CORS using middleware in the Program.cs file. This approach allows you to apply a CORS policy to all endpoints or specific endpoints2.

**Using Endpoint Routing:** CORS can be configured using endpoint routing, which provides more granular control over which endpoints support CORS.

**Using the `[EnableCors]` Attribute:** You can use the [EnableCors] attribute to enable CORS for specific controllers or actions.

# Example Configuration
Here's an example of how to enable CORS using middleware in Program.cs:

```cs
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddCors(options =>
{
    options.AddPolicy("MyAllowSpecificOrigins", policy =>
        policy.WithOrigins("http://example.com", "http://www.contoso.com"));
});
builder.Services.AddControllers();
var app = builder.Build();
app.UseCors("MyAllowSpecificOrigins");
app.MapControllers();
app.Run();

```