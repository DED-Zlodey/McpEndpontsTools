![Static Badge](https://img.shields.io/badge/MCP%20SDK-0.2.0%20preview.1-%239553E9?logo=dotnet)
![Static Badge](https://img.shields.io/badge/MCP%20Endpoints%20Tools-v1.0.12%20alpha-%239553E9?logo=dotnet)

# MCP Endpoints Tools

Library for ASP.NET Core Web API, which automatically turns each controller method into a tool for the MCP
server.

Under the hood, MCP Endpoints Tools uses
the [Model Context Protocol C# SDK](https://github.com/modelcontextprotocol/csharp-sdk "Model Context Protocol C# SDK")
for working with tools.

> [!NOTE]
> This project is in alpha version! Various errors are possible. Please write to the issue for errors and suggestions on library expansion.

## Description

MCP Endpoints Tools scans the application build, finds all controllers and their public methods marked with HTTP
attributes, and registers them as Model Context Protocol (MCP) tools. In this case, XML comments from the assembly are
used to fill in the description (summary) of the tools.

## Features

* Automatic registration of all controller methods as MCP tools and resources
* Support for method exclusion via the `[McpIgnore]` attribute
* Automatic addition of the tool description from the XML comments of the assembly via the 'XmlCommentsProvider`
* Flexible configuration via `ServerOptions` (path, name, description, version, XML path)
* Easy integration into 'IServiceCollection` and `IEndpointRouteBuilder' via extensions `ServiceCollectionExtensions`
  and `EndpointRouteBuilderExtensions`

## Installation

1. Add the `McpEndpointsTools` project to your solution or connect via NuGet (if there is a package):

   ```bash
   dotnet add package McpEndpointsTools --version 1.0.12-alpha
   ```

2. In the file `Program.cs` (or `Startup.cs`), register the services and mapping:

   ```csharp
     using McpEndpointsServer.Extensions;

     var builder = WebApplication.CreateBuilder(args);

     // MCP Server registration and controller scanning
     builder.Services.AddMcpEndpointsServer(opts =>
     {
            opts.PipelineEndpoint = "/mcp"; // path for the HTTP pipeline
            opts.ServerName = "My MCP Server"; // server name
            opts.ServerDescription = "API for MCP"; // description
            opts.ServerVersion = "1.2.3"; // version
            opts.XmlCommentsPath = "MyApp.xml "; // path to the XML documentation file
            opts.HostUrl = "https://api.mysite "; // base URL
     });
   ```


3. **Enable XML documentation generation** in your `.csproj` project file:
   ```xml
   <PropertyGroup>
     <GenerateDocumentationFile>true</GenerateDocumentationFile>
     <NoWarn>$(NoWarn);1591</NoWarn>
   </PropertyGroup>
   ```

4. Set up routing in the same or another location:

   ```csharp
   var app = builder.Build();

   app.MapControllers(); // regular controllers
   app.MapMcpEndpointsServer(); // MCP endpoints (HTTP stream & SSE)

   app.Run();
   ```
   **Follow the registration procedure**

## Attributes

* `McpIgnoreAttribute` It is placed above the controller method to exclude it from the list of generated MCP tools.
* `Authorize` Methods and controllers marked with the Authorize attribute will not be added to the tools. In future
  versions, support for authentication authorization may be added.

Here is an example of an attribute over a controller method

```csharp
    /// <summary>
    /// Provides clothing advice based on the current temperature in Celsius.
    /// </summary>
    /// <param name="tempC">The temperature in Celsius for which clothing advice is needed.</param>
    /// <returns>
    /// A string containing clothing advice suitable for the specified temperature.
    /// </returns>
    [HttpGet("GetClothingAdvice")]
    [McpIgnore]
    public IActionResult GetClothingAdvice([FromQuery] double tempC)
    {
        if (tempC >= 25) return Ok("Put on a T-shirt and shorts.");
        if (tempC >= 15) return Ok("A light jacket will do.");
        if (tempC >= 5) return Ok("I need a coat.");
        return Ok("It's very cold — keep warm!");
    }
```

If you mark the entire controller with this attribute, all controller methods will be ignored by the MCP server.

### Example app

See the sample application on ASP .NET Core is available via
the [link](https://github.com/DED-Zlodey/McpEndpontsTools/tree/master/WebApiExample "WebApiExample")


### Example config for IDEs

#### VS Code global

   ```json
   {
       "workbench.startupEditor": "none",
       "explorer.confirmDelete": false,
       "mcp": {
           "servers": {
               "test-mcp": {
                   "url": "http://localhost:5220/mcp"
               },
               "API-helper": {
                   "url": "https://api.il2-expert.ru/mcp"
               }
           }
       },
       "explorer.confirmDragAndDrop": false,
       "chat.editing.confirmEditRequestRetry": false
   }
   ```
#### Cursor global
   ```json
   {
       "mcpServers": {
           "my-mcp-server": {
               "url": "http://localhost:5258/mcp",
               "type": "http"
           }
       }
   }
   ```


#### Live demo
- The demo API application is deployed at: [https://api.il2-expert.ru/mcp/resources](https://api.il2-expert.ru/mcp/resources "Demo")
- To test connecting the MCP IDE client to the endpoint: https://api.il2-expert.ru/mcp


## License

MIT License. See the LICENSE file for details.
