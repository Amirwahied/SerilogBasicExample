# Serilog
Serilog is a diagnostic logging library for .NET applications. It is easy to set up, has a clean API, and runs on all recent .NET platforms. While it's useful even in the simplest applications, Serilog's support for structured logging shines when instrumenting complex, distributed, and asynchronous applications and systems.

## What I Do ?
In this basic example, I used the Serilog library for logging to both a file and SQL Server, Using C# Language in ASP.Net Core API Project


### Install and configuration

#### Needed Packages

1-	Serilog.AspNetCore  				       (Basic Library)

2-	Serilog.Settings.Configuration		 (Basic Library)

3-	Serilog.Sinks.File			        	 (For logging on file)

4-	Serilog.Sinks.MSSSqlServer		     (For logging on SQL Server DB)(In addition to Entity Framework Core Libraries)

#### Serilog Configuration in Program.cs

```
//Setup logger configuration from appsettings.json
var Logger = new LoggerConfiguration()
                .ReadFrom
                .Configuration(builder.Configuration) 
                .CreateLogger();

//Add logger after configured
builder.Logging.AddSerilog(Logger);
```

#### Serilog Configuration in appsettings.json

More configuration like (libraries you want to use ex(File, SQL Server DB), Minimum level of logging)

```
"Serilog": {
   "Using": [ "Serilog.Sinks.File", "Serilog.Sinks.MongoDB","Serilog.Sinks.MSSqlServer" ],
   "MinimumLevel": {
     "Default": "Information",
     "Override": {
       "Microsoft": "Error",
       "MongoDB": "Error",
       "System": "Error"
     }
   }
 }
```

Now, tell logger where to add logs and adjust arguments of the source.

In case of Sql server, there is a property to auto generate table in the database that logs will saved in ("autoCreateSqlTable": true), just write the table name and serilog will create it.

```
 "WriteTo": [
   {
     "Name": "File",
     "Args": {
       //The output filename will be "Logger-20240124.log" as serilog will add created date in each log file
       "path": "../Logs/Logger-.log",
       //Create a new file daily with today date
       "rollingInterval": "Day",
       //Template of log message
       "outputTemplate": "{Timestamp:yyyy-MM-dd HH:mm:ss.fff zzz} [{Level:u3}] {Message:lj}{NewLine}{Exception}"
     }
   },
   {
     "Name": "MSSqlServer",
     "Args": {
       //Add LogDB connection string
       "connectionString": "",
       //The name of table of logs (you can create it or leave this task to serilog to create it)
       "tableName": "Logs",
       //Here serilog will create it automatically
       "autoCreateSqlTable": true
     }
   }
 ]
```

#### Configure SQL Database in Program.cs

```
//Database Configuration where connection string is in appsetting.json
builder.Services.AddDbContext<ApplicationDBContext>(opt => opt.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

```

#### Now,Let's Try !!

##### Inject the ILogger interface in the constructor of logger

```
 public WeatherForecastController(ILogger<WeatherForecastController> logger)
 {
    _logger = logger;
 }
```

##### Throw an exception in try catch and log it in any endpoint

```
  try
  {
      throw new ArgumentNullException("The Arguments is null"); // Message
  }
  catch (Exception ex)
  {
      _logger.LogError(ex, ex.Message); 
  }

```
