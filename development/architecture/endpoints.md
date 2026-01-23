# Endpoints

There are countless, similar definitions for Endpoints in literature and on the web (e.g. on [smartbear.com](https://smartbear.com/learn/performance-monitoring/api-Endpoints/) or [techtarget.com](https://www.techtarget.com/searchapparchitecture/definition/RESTful-API)).
An Endpoint in the MORYX-Framework acts in line with the general concept these definitions outline.
Therefore, this document merely reframes these definitions using the terms introduced in the `MORYX Design Guidelines`.
In addition, this article points out how to integrate an Endpoint in the MORYX-Framework with some Do's and Dont's along the way.

## MORYX Endpoints

In a MORYX Application, an Endpoint exposes a Module's Facade externally.
While the Facade generally defines what the Endpoint can expose, each Endpoint translates this to a different technology and thereby to its own way of accessing it.
Thereby, a method from a Module and the logic it implements might be accessible via an MQTT broker as well as an HTTP request.

## Best Practices

If an Endpoint exists, which provides more functionality than you need, use it.

If an Endpoint based on a different technology exists, use it.
(Unless it's not possible to switch to this technology of course, in this case follow the subsequent *Best Practices*)

A new Endpoint ideally covers all functionality a Facade offers, so that duplicate effort in development and maintenance can be avoided.
Of course, there are limits set by the reasonable effort to design an Endpoint or it might not even be possible to replicate a facade as an Endpoint with a certain technology.

An Endpoint should be targeted toward a specific Module.
This means DON'T create a God-Endpoint that aggregates all facades in existence.
(Each Module is its own entity for a reason. Don't screw up the modularity others put hard work into.)
If the reference to an additional Facade is required, however, you can make use of these references (e.g. for transformation, like Ids to Objects, or for timing behavior).

An Endpoint should contain as little logic as possible.

- If the Endpoint engages in an interaction with an IT-System actively (and holds the required logic for it), build an adapter.
(The chances that you can reuse it for a different IT-System are quite small)
- If you only want a subset of the Information the Endpoint provides, consider filtering it in the consumer (e.g. in the UI)
- If your Endpoint provides its own logic to the MORYX-Framework make it a Module - you might find a use for the Facade internally as well.
(This specifically includes cases in which you need (1) state-dependent behavior, (2) configurations, (3) a database, or (4) a Lifecycle)
- Processing within your Endpoint should be interaction-based.
(This means each interaction through the Endpoint is a flow of information based on filtering and transformations)

## Example

This section provides an example for a REST-Endpoint on the [Notification Publisher Facade](https://github.com/PHOENIXCONTACT/MORYX-Framework/blob/future/docs/articles/module-notifications/index.md). The REST-Endpoint makes it possible to access notifications via simple HTTP requests.

### Folder Structure

Besides the project for the Module, a new project called `Moryx.Notifications.Publisher.Endpoints` under *src/Moryx.Notifications.Publisher.Endpoints* is created.
Simply use the .Net Core library template and add a reference to the project or package that contains the API definition of the Module.
In this case, that is the [Moryx.Notifications Package](https://www.nuget.org/packages/Moryx.Notifications/).

### The REST-Endpoint

Adding the Rest-Endpoint is rather simple.
Start by adding a `NotificationPublisherController` class to the library project.

```csharp
[ApiController]
[Route("api/moryx/notifications/")]
[Produces("application/json")]
public class NotificationPublisherController : ControllerBase
{
    private readonly INotificationPublisher _notificationPublisher;

    public NotificationPublisherController(INotificationPublisher notificationPublisher)
        => _notificationPublisher = notificationPublisher;

    [HttpGet]
    public ActionResult<NotificationModel[]> GetAll()
    {
        return _notificationPublisher.GetAll().Select(Converter.ToModel).ToArray();
    }

    [HttpGet("{guid}")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public ActionResult<NotificationModel> Get(Guid guid)
    {
        var notification = _notificationPublisher.Get(guid);
        if (notification == null)
            return NotFound();

        return Converter.ToModel(notification);
    }

    [HttpPost("{guid}/acknowledge")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public ActionResult Acknowledge(Guid guid)
    {
        var notification = _notificationPublisher.Get(guid);
        if (notification == null)
            return NotFound();

        _notificationPublisher.Acknowledge(notification);
        return Ok();
    }
}
```

This controller gets the NotificationPublisher via dependency injection in the constructor.
For each of the callable methods in the `INotificationPublisher` interface, the controller provides a respective method.
These methods define the behavior on Get or Post requests under the defined route.

To add this Endpoint to your runtime, just make sure the [AddControllers()](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.mvcservicecollectionextensions.addcontrollers?view=aspnetcore-6.0) method is called in your `Startup.cs` and your project is referenced.

#### Generating a nice Open API definition

Different tools can generate an API definition based on the code in the controller.
A common approach is to use [Swagger](https://duckduckgo.com/?q=swagger&ia=web) for this task (besides its usage for manual testing of the Endpoint).

In order to get a precise documentation for your API, there are three additional specifications that can be added.

First, the class attribute in you controller `[Produces("application/json")]` specifies which type of response to expect from the Endpoint, in this case a JSON response. (Note that this attribute can also be defined on each method individually.)

Second, and this is not mandatory just helpful for human readability, replace the auto generated way of adding controllers to the application

```csharp
// Startup.cs
services.AddControllers();
```

with a slightly customized configuration of the same line. Thereby, you get nicely readable serializations of your enums.

```csharp
// Startup.cs
services.AddControllers().AddJsonOptions(jo => jo.JsonSerializerOptions.Converters.Add(new JsonStringEnumConverter()));
```

A similar thing can be done for the SignalR endpoint described in the following section by adding

```csharp
// Startup.cs
services.AddSignalR().AddJsonProtocol(options =>
{
    options.PayloadSerializerOptions.Converters
        .Add(new JsonStringEnumConverter());
});
```

to your ConfigureServices method.

Lastly, your can tell Swagger to use the method names as Ids for the operations found in the API definition.
Again, this is but a single line you need to add to the ConfigureServices method

```csharp
// Startup.cs
services.AddSwaggerGen(c =>
{
    c.CustomOperationIds(api => ((ControllerActionDescriptor)api.ActionDescriptor).MethodInfo.Name);
});
```
