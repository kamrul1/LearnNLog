# Custom logging with NLog

>Simplest of example from following NLog exercise:\
[ASP.NET Core Web API – Logging With NLog](https://code-maze.com/net-core-web-development-part3/)


There are alternative loggers such as Serilog, which is I have found to 
be more widely used and offer more options (Provided Sinks) out of the box.

## Step 1:
Create 3 projects:
- AccountOwnerServer web api project
  - references LoggerService
  
- Contracts library project

- LoggerService library project
  - references Contracts

## Creating the interface and installing NLog

1. Create for 4 logging interface for NLog

```csharp
namespace Contracts
{
    public interface ILoggerManager
    {
        void LogInfo(string message);
        void LogWarn(string message);
        void LogDebug(string message);
        void LogError(string message);
    }
}
```

2. Add NLog.Extensions.Logging nuget to LoggerService
3. Create LoggerManager in LoggerService, implement ILoggerManager interface
```csharp
using Contracts;
using NLog;

namespace LoggerService
{
    public class LoggerManager : ILoggerManager
    {
        private static ILogger logger = LogManager.GetCurrentClassLogger();

        public void LogDebug(string message)
        {
            logger.Debug(message);
        }

        public void LogError(string message)
        {
            logger.Error(message);
        }

        public void LogInfo(string message)
        {
            logger.Info(message);
        }

        public void LogWarn(string message)
        {
            logger.Warn(message);
        }
    }
}
```



4. Create nlog.config file to specify logging configurations
```xml
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      autoReload="true"
      internalLogLevel="Trace"
      internalLogFile="c:internal_logs\internallog.txt">

  <targets>
    <target name="logfile" xsi:type="File"
            fileName="c:/repo/LearnNLog/Logs/${shortdate}_logfile.txt"
            layout="${longdate} ${level:uppercase=true} ${message}"/>
  </targets>

  <rules>
    <logger name="*" minlevel="Debug" writeTo="logfile" />
  </rules>
</nlog>

```

5. Configuring logger service for Logging Messages
```csharp
public Startup(IConfiguration configuration)
{
    LogManager.LoadConfiguration(String.Concat(Directory.GetCurrentDirectory(), "/nlog.config"));
    Configuration = configuration;
}
```

6. Inject ILoggerManager in Startup class in AccountOwnerServer

I used an ServiceExtension, but could have just added it to service directly.

```csharp
namespace AccountOwnerServer
{
    public static class ServiceExtensions
    {
        public static void ConfigureLoggerService(this IServiceCollection services)
        {
            services.AddSingleton<ILoggerManager, LoggerManager>();
        }
    }
}
```

In our Startup.cs, ConfigureService method
```csharp
services.ConfigureLoggerService();
```

7. User the LoggerManager in our controller

```csharp

[ApiController]
[Route("[controller]")]
public class WeatherForecastController : ControllerBase
{
    private readonly ILoggerManager _logger;

    public WeatherForecastController(ILoggerManager logger)
    {
        _logger = logger;
    }

    [HttpGet]
    public IEnumerable<string> Get()
    {
        _logger.LogInfo("Here is info message from the controller.");
        _logger.LogDebug("Here is debug message from the controller.");
        _logger.LogWarn("Here is warn message from the controller.");
        _logger.LogError("Here is error message from the controller.");

        return new string[] { "value1", "value2" };
    }
}

```

After calling the endpoint for http://localhost:5000/weatherforecast check the NLog files into the two location.
They should update with logs:
```
2021-10-24 03:33:59.7793 INFO Here is info message from the controller.
2021-10-24 03:33:59.7793 DEBUG Here is debug message from the controller.
2021-10-24 03:33:59.7793 WARN Here is warn message from the controller.
2021-10-24 03:33:59.7793 ERROR Here is error message from the controller.
2021-10-24 03:39:00.9208 INFO Here is info message from the controller.
2021-10-24 03:39:00.9571 DEBUG Here is debug message from the controller.
2021-10-24 03:39:00.9673 WARN Here is warn message from the controller.
2021-10-24 03:39:00.9673 ERROR Here is error message from the controller.
```




