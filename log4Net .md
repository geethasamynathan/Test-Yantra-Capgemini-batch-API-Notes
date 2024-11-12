# Log4Net
Log4net is a popular open-source logging library for .NET applications, part of the Apache Logging Services project. It provides a flexible way to write logs to various output targets like files, consoles, databases, emails, etc.

# Why Use Log4net?
**Structured Logging:** Captures logs in a structured format (key-value pairs or JSON objects) rather than plain text, making it easier to query and analyze.

**Flexibility:** Supports multiple logging targets (appenders) and allows for easy configuration changes.

**Debugging and Troubleshooting:** Helps developers identify and fix bugs by providing detailed context about what the application was doing at a specific point in time.

**Monitoring and Auditing:** Records actions for security purposes and ensures compliance with regulations.

can you give same example in vs 2022
Sure! Here’s how you can set up Log4net in an ASP.NET Core Web API project using Visual Studio 2022 with .NET 6.0.

## Step-by-Step Example
**Step 1:** Create a New ASP.NET Core Web API Project
Open Visual Studio 2022.

Create a New Project:

Go to File > New > Project.

Select ASP.NET Core Web API and click Next.

Name your project (e.g., BikeStoreAPI), choose a location, and click Create.

Select .NET 6.0 (Long-term support) and click Create.

**Step 2:** Install the Log4net Package
Install Log4net:

Open the NuGet Package Manager Console (Tools > NuGet Package Manager > Package Manager Console).

Run the following command to install Log4net:
`Install-Package log4net`

Step 3: Create a Log4net Configuration File
Add Log4net Configuration File:

Right-click on your project in Solution Explorer, select Add > New Item.

Choose XML File and name it log4net.config.

## Configure Log4net:

Add the following XML configuration to log4net.config:

```xml
<log4net>
    <appender name="RollingFile" type="log4net.Appender.RollingFileAppender">
        <file value="logs/app.log" />
        <appendToFile value="true" />
        <rollingStyle value="Date" />
        <datePattern value="yyyy-MM-dd" />
        <maxSizeRollBackups value="10" />
        <layout type="log4net.Layout.PatternLayout">
            <conversionPattern value="%date [%thread] %-5level %logger - %message%newline" />
        </layout>
    </appender>
    <root>
        <level value="ALL" />
        <appender-ref ref="RollingFile" />
    </root>
</log4net>

```
## Explanation
### Appender Configuration
<appender name="RollingFile" type="log4net.Appender.RollingFileAppender">
name: The name of this appender is "RollingFile".

type: The type of appender used is RollingFileAppender, which writes log messages to a file and rolls the file based on specified criteria.

### File Settings

<file value="logs/app.log" />
value: Specifies the path and name of the log file. In this case, logs are saved in logs/app.log.

### Appending Settings
<appendToFile value="true" />
value: When set to true, new log messages are appended to the existing log file. If set to false, the log file is overwritten each time the application starts.

### Rolling Style
<rollingStyle value="Date" />
value: Specifies that the log file will roll (create a new file) based on the date.

### Date Pattern
<datePattern value="yyyy-MM-dd" />
value: Defines the date pattern used in the rolled file names. For example, app.log.2023-12-10 for December 10th, 2023.

### Max Size Roll Backups
<maxSizeRollBackups value="10" />
value: Limits the number of backup files to 10. Older files beyond this limit will be deleted.

### Layout
<layout type="log4net.Layout.PatternLayout">
Type: The layout type used is PatternLayout, which formats the log messages

### Conversion Pattern

<conversionPattern value="%date [%thread] %-5level %logger - %message%newline" />
**value:** Defines the pattern for formatting log messages.

**%date:** The date and time of the log entry.

**[%thread]:** The thread that generated the log entry.

**%-5level:** The log level (e.g., INFO, WARN, ERROR) padded to 5 characters.

**%logger:** The name of the logger that generated the log entry.

**%message:** The log message.

**%newline:** Inserts a newline after the log message.

### Root Logger Configuration

<root>
    <level value="ALL" />
    <appender-ref ref="RollingFile" />
</root>
<level value="ALL" />: Sets the logging level to ALL, meaning all levels of log messages (DEBUG, INFO, WARN, ERROR, FATAL) will be captured.

<appender-ref ref="RollingFile" />: Associates the root logger with the "RollingFile" appender, directing log messages to the specified log file.


**Set Log4net Config File as Copy Always:**

Right-click log4net.config in Solution Explorer.

Select Properties.

Set Copy to Output Directory to Copy always.

**Step 4:** Configure Log4net in Your Application
Configure Log4net in Program.cs:

Open Program.cs and add the following configuration to set up Log4net:

```cs
using log4net;
using log4net.Config;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Hosting;
using System.IO;

public class Program
{
    public static void Main(string[] args)
    {
        var logRepository = LogManager.GetRepository(System.Reflection.Assembly.GetEntryAssembly());
        XmlConfigurator.Configure(logRepository, new FileInfo("log4net.config"));
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```

Add Log4net to Your Controllers:

Open an existing controller or create a new one (e.g., BikesController.cs).

Add logging using Log4net:

```cs
using Microsoft.AspNetCore.Mvc;
using log4net;

[Route("api/[controller]")]
[ApiController]
public class BikesController : ControllerBase
{
    private static readonly ILog log = LogManager.GetLogger(typeof(BikesController));

    [HttpGet]
    public IActionResult Get()
    {
        log.Info("BikesController Get method called");
        return Ok(new string[] { "Bike1", "Bike2" });
    }
}

```

Step 5: Run and Test the Application
**Run Your Application:**

Press F5 or click the green "Start" button in Visual Studio to start the application.

**Verify Logging:**

Make a request to your API endpoint (e.g., https://localhost:5001/api/bikes).

Check the logs directory in your project root for the app.log file to see the logged messages.

# Explanation
**Log4net Configuration:** The log4net.config file specifies how logs are written, including the file location, rolling style, and log format.

**Program Configuration:** The Program.cs file sets up Log4net by configuring the logger repository with the log4net.config file.

Step 5: Run and Test the Application
Run Your Application:

Press F5 or click the green "Start" button in Visual Studio to start the application.

Verify Logging:

Make a request to your API endpoint (e.g., https://localhost:5001/api/bikes).

Check the logs directory in your project root for the app.log file to see the logged messages.

Explanation
Log4net Configuration: The log4net.config file specifies how logs are written, including the file location, rolling style, and log format.

Program Configuration: The Program.cs file sets up Log4net by configuring the logger repository with the log4net.config file.

Logging in Controllers: The BikesController demonstrates how to log messages using Log4net. The BikesController demonstrates how to log messages using Log4net.