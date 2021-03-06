# Checkout Heartbeat

A fluent api to run health checks in background for your console application.

Metrics are based on [AppMetrics](https://www.app-metrics.io/web-monitoring/aspnet-core/) and the background service implementation is mainly based on [this](https://blogs.msdn.microsoft.com/cesardelatorre/2017/11/18/implementing-background-tasks-in-microservices-with-ihostedservice-and-the-backgroundservice-class-net-core-2-x/) article

## Nuget

The Nuget package is available on our internal [MyGet](https://www.myget.org/feed/ckotech/package/nuget/Checkout.Heartbeat)

## Usage

The main actor in our heartbeat process is the Health Monitor Runner, 
You can simply instantiate one in the following way:

```c#

var runner = new HealthMonitorRunner();

```

or if you alreday have your own StructureMap IContainer available you can directly inject it


```c#

var runner = new HealthMonitorRunner(myContainer);

```

__Configuration__

Option 1 - Programmatic:

```c#

runner.Configure(options =>
{
   options.Frequency = TimeSpan.FromSeconds(3);
   options.Timeout = TimeSpan.FromSeconds(3);
   options.PrivateMemorySizeLimit = 100; //bytes
   options.VirtualMemorySizeLimit = 200; //bytes
   options.PhysicalMemorySizeLimit = 300; //bytes
   options.PingChecks = new[]
   {
       "google.com",
       "checkout.com"
   };
   options.HttpGetChecks = new[]
   {
       new Uri("https://github.com/"),
       new Uri("https://checkout.com/")
   };
});
                 
```

Option 2 - via Configuration

```c#

var configuration = new ConfigurationBuilder()
.SetBasePath(Directory.GetCurrentDirectory())
.AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
.Build();

runner.ConfigureFrom(configuration);

```

*and in your appsettings.json*


```js
{
  ...
  ...
  "HealthCheckMonitor": {
    "Timeout": 10000, //in milliseconds
    "Frequency": 3000, //in milliseconds
    "PrivateMemorySizeLimit": 100, //bytes
    "VirtualMemorySizeLimit": 200, //bytes
    "PhysicalMemorySizeLimit": 300, //bytes
    "PingChecks": [
      "google.com",
      "checkout.com"
    ],
    "HttpGetChecks": [
      "https://github.com/",
      "https://checkout.com/"
    ]
  }
  ...
  ...
}
```

__Defining your custom action on health event__

```c#

runner.OnHealthStatus(status =>
{
    _logger.Information(JsonConvert.SerializeObject(status, Formatting.Indented));
})
.Start();

```

__Starting the Monitor__

```c#

runner.Start();

```

## Custom Health Check

```c#

 public class MyCustomHealthCheck : HealthCheck
 {
     public MyCustomHealthCheck() : base("My Custom - Health Check") { }

     protected override ValueTask<HealthCheckResult> CheckAsync(CancellationToken cancellationToken = default(CancellationToken))
     {
         if (DateTime.UtcNow.Second <= 20)
         {
             return new ValueTask<HealthCheckResult>(HealthCheckResult.Degraded());
         }

         if (DateTime.UtcNow.Second >= 40)
         {
             return new ValueTask<HealthCheckResult>(HealthCheckResult.Unhealthy());
         }

         return new ValueTask<HealthCheckResult>(HealthCheckResult.Healthy());
     }
 }

```
