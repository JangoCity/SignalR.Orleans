# Configuration

## Silo
We need to configure the Orleans Silo with the below:
* Use `.UseSignalR()` on `ISiloHostBuilder`.

***Example***
```cs
var silo = new SiloHostBuilder()
    .UseSignalR()
    .Build();

await silo.StartAsync();
```

### Configure Silo Storage Provider and Grain Persistance
Optional configuration to override the default implementation for both providers which by default are set as `Memory`.

***Example***
```cs
.UseSignalR(cfg =>
{
    cfg.ConfigureBuilder = (builder, config) =>
    {
        builder
            .AddMemoryGrainStorage(config.PubSubProvider)
            .AddMemoryGrainStorage(config.StorageProvider);
    };
})
```

## Client
Now your SignalR application needs to connect to the Orleans Cluster by using an Orleans Client:
* Use `.UseSignalR()` on `IClientBuilder`.

***Example***
```cs
var client = new ClientBuilder()
    .UseSignalR()
    .Build();

await client.Connect();
```

Somewhere in your `Startup.cs`:
* Use `.AddSignalR()` on `IServiceCollection` (this is part of `Microsoft.AspNetCore.SignalR` nuget package).
* Use `AddOrleans()` on `.AddSignalR()`.

***Example***
```cs
public void ConfigureServices(IServiceCollection services)
{
    ...
    services
        .AddSignalR()
        .AddOrleans();
    ...
}
```
Great! Now you have SignalR configured and Orleans SignalR backplane built in Orleans!

# Features
## Hub Context
`HubContext` gives you the ability to communicate with the client from orleans grains (outside the hub).

Sample usage: Receiving server push notifications from message brokers, web hooks, etc. Ideally first update your grain state and then push signalr message to the client.

### Example: 
```cs
public class UserNotificationGrain : Grain<UserNotificationState>, IUserNotificationGrain
{
	private HubContext<IUserNotificationHub> _hubContext;

	public override async Task OnActivateAsync()
	{
		_hubContext = GrainFactory.GetHub<IUserNotificationHub>();
		// some code...
		await _hubContext.User(this.GetPrimaryKeyString()).SendSignalRMessage("Broadcast", State.UserNotification);
	}
}
```