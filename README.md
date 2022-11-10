Tutorial: Create a web API with ASP.NET Core
Article
11/08/2022
63 minutes to read
47 contributors
By Rick Anderson and Kirk Larkin

This tutorial teaches the basics of building a web API using a database.

In this tutorial, you learn how to:

Create a web API project.
Add a model class and a database context.
Scaffold a controller with CRUD methods.
Configure routing, URL paths, and return values.
Call the web API with http-repl.
At the end, you have a web API that can manage "to-do" items stored in a database.

Overview
This tutorial creates the following API:

API	Description	Request body	Response body
GET /api/todoitems	Get all to-do items	None	Array of to-do items
GET /api/todoitems/{id}	Get an item by ID	None	To-do item
POST /api/todoitems	Add a new item	To-do item	To-do item
PUT /api/todoitems/{id}	Update an existing item  	To-do item	None
DELETE /api/todoitems/{id}    	Delete an item    	None	None
The following diagram shows the design of the app.

The client is represented by a box on the left. It submits a request and receives a response from the application, a box drawn on the right. Within the application box, three boxes represent the controller, the model, and the data access layer. The request comes into the application's controller, and read/write operations occur between the controller and the data access layer. The model is serialized and returned to the client in the response.

Prerequisites
Visual Studio
Visual Studio Code
Visual Studio for Mac
Visual Studio Code
C# for Visual Studio Code (latest version)
.NET 6.0 SDK
The Visual Studio Code instructions use the .NET CLI for ASP.NET Core development functions such as project creation. You can follow these instructions on macOS, Linux, or Windows and with any code editor. Minor changes may be required if you use something other than Visual Studio Code.

Create a web project
Visual Studio
Visual Studio Code
Visual Studio for Mac
Open the integrated terminal.

Change directories (cd) to the folder that will contain the project folder.

Run the following commands:

.NET CLI

Copy
dotnet new webapi -o TodoApi
cd TodoApi
dotnet add package Microsoft.EntityFrameworkCore.InMemory
code -r ../TodoApi
These commands:

Create a new web API project and open it in Visual Studio Code.
Add a NuGet package that is needed for the next section.
When a dialog box asks if you want to add required assets to the project, select Yes.

 Note

For guidance on adding packages to .NET apps, see the articles under Install and manage packages at Package consumption workflow (NuGet documentation). Confirm correct package versions at NuGet.org.

Test the project
The project template creates a WeatherForecast API with support for Swagger.

Visual Studio
Visual Studio Code
Visual Studio for Mac
Trust the HTTPS development certificate by running the following command:

.NET CLI

Copy
dotnet dev-certs https --trust
The preceding command doesn't work on Linux. See your Linux distribution's documentation for trusting a certificate.

The preceding command displays the following dialog, provided the certificate was not previously trusted:

Security warning dialog

Select Yes if you agree to trust the development certificate.

See Trust the ASP.NET Core HTTPS development certificate for more information.

For information on trusting the Firefox browser, see Firefox SEC_ERROR_INADEQUATE_KEY_USAGE certificate error.

Run the app:

Press Ctrl+F5.
At the Select environment prompt, choose .NET Core.
Select Add Configuration > .NET: Launch a local .NET Core Console App.
In the configuration JSON:
Replace <target-framework> with net6.0.
Replace <project-name.dll> with TodoApi.dll.
Press Ctrl+F5.
In the Could not find the task 'build' dialog, select Configure Task.
Select Create tasks.json file from template.
Select the .NET Core task template.
Press Ctrl+F5.
In a browser, navigate to https://localhost:<port>/swagger, where <port> is the randomly chosen port number displayed in the output.

The Swagger page /swagger/index.html is displayed. Select GET > Try it out > Execute. The page displays:

The Curl command to test the WeatherForecast API.
The URL to test the WeatherForecast API.
The response code, body, and headers.
A drop-down list box with media types and the example value and schema.
If the Swagger page doesn't appear, see this GitHub issue.

Swagger is used to generate useful documentation and help pages for web APIs. This tutorial focuses on creating a web API. For more information on Swagger, see ASP.NET Core web API documentation with Swagger / OpenAPI.

Copy and paste the Request URL in the browser: https://localhost:<port>/weatherforecast

JSON similar to the following example is returned:

JSON

Copy
[
    {
        "date": "2019-07-16T19:04:05.7257911-06:00",
        "temperatureC": 52,
        "temperatureF": 125,
        "summary": "Mild"
    },
    {
        "date": "2019-07-17T19:04:05.7258461-06:00",
        "temperatureC": 36,
        "temperatureF": 96,
        "summary": "Warm"
    },
    {
        "date": "2019-07-18T19:04:05.7258467-06:00",
        "temperatureC": 39,
        "temperatureF": 102,
        "summary": "Cool"
    },
    {
        "date": "2019-07-19T19:04:05.7258471-06:00",
        "temperatureC": 10,
        "temperatureF": 49,
        "summary": "Bracing"
    },
    {
        "date": "2019-07-20T19:04:05.7258474-06:00",
        "temperatureC": -1,
        "temperatureF": 31,
        "summary": "Chilly"
    }
]
Update the launchUrl
In Properties\launchSettings.json, update launchUrl from "swagger" to "api/todoitems":

JSON

Copy
"launchUrl": "api/todoitems",
Because Swagger will be removed, the preceding markup changes the URL that is launched to the GET method of the controller added in the following sections.

Add a model class
A model is a set of classes that represent the data that the app manages. The model for this app is a single TodoItem class.

Visual Studio
Visual Studio Code
Visual Studio for Mac
Add a folder named Models.

Add a TodoItem.cs file to the Models folder with the following code:

C#

Copy
namespace TodoApi.Models
{
    public class TodoItem
    {
        public long Id { get; set; }
        public string? Name { get; set; }
        public bool IsComplete { get; set; }
    }
}
The Id property functions as the unique key in a relational database.

Model classes can go anywhere in the project, but the Models folder is used by convention.

Add a database context
The database context is the main class that coordinates Entity Framework functionality for a data model. This class is created by deriving from the Microsoft.EntityFrameworkCore.DbContext class.

Visual Studio
Visual Studio Code
Visual Studio for Mac
Add a TodoContext.cs file to the Models folder.
Enter the following code:

C#

Copy
using Microsoft.EntityFrameworkCore;
using System.Diagnostics.CodeAnalysis;

namespace TodoApi.Models
{
    public class TodoContext : DbContext
    {
        public TodoContext(DbContextOptions<TodoContext> options)
            : base(options)
        {
        }

        public DbSet<TodoItem> TodoItems { get; set; } = null!;
    }
}
Register the database context
In ASP.NET Core, services such as the DB context must be registered with the dependency injection (DI) container. The container provides the service to controllers.

Update Program.cs with the following code:

C#

Copy
using Microsoft.EntityFrameworkCore;
using TodoApi.Models;


var builder = WebApplication.CreateBuilder(args);

// Add services to the container.

builder.Services.AddControllers();
builder.Services.AddDbContext<TodoContext>(opt =>
    opt.UseInMemoryDatabase("TodoList"));
//builder.Services.AddSwaggerGen(c =>
//{
//    c.SwaggerDoc("v1", new() { Title = "TodoApi", Version = "v1" });
//});

var app = builder.Build();

// Configure the HTTP request pipeline.
if (builder.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
    //app.UseSwagger();
    //app.UseSwaggerUI(c => c.SwaggerEndpoint("/swagger/v1/swagger.json", "TodoApi v1"));
}

app.UseHttpsRedirection();

app.UseAuthorization();

app.MapControllers();

app.Run();
The preceding code:

Removes the Swagger calls.
Removes unused using directives.
Adds the database context to the DI container.
Specifies that the database context will use an in-memory database.
Scaffold a controller
Visual Studio
Visual Studio Code
Visual Studio for Mac
Make sure that all of your changes so far are saved.

Run the following commands from the project folder, that is, the TodoApi folder:

.NET CLI

Copy
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet tool install -g dotnet-aspnet-codegenerator
dotnet-aspnet-codegenerator controller -name TodoItemsController -async -api -m TodoItem -dc TodoContext -outDir Controllers
The preceding commands:

Add NuGet packages required for scaffolding.
Install the scaffolding engine (dotnet-aspnet-codegenerator).
Scaffold the TodoItemsController.
The generated code:

Marks the class with the [ApiController] attribute. This attribute indicates that the controller responds to web API requests. For information about specific behaviors that the attribute enables, see Create web APIs with ASP.NET Core.
Uses DI to inject the database context (TodoContext) into the controller. The database context is used in each of the CRUD methods in the controller.
The ASP.NET Core templates for:

Controllers with views include [action] in the route template.
API controllers don't include [action] in the route template.
When the [action] token isn't in the route template, the action name is excluded from the route. That is, the action's associated method name isn't used in the matching route.

Update the PostTodoItem create method
Update the return statement in the PostTodoItem to use the nameof operator:

C#

Copy
[HttpPost]
public async Task<ActionResult<TodoItem>> PostTodoItem(TodoItem todoItem)
{
    _context.TodoItems.Add(todoItem);
    await _context.SaveChangesAsync();

    //return CreatedAtAction("GetTodoItem", new { id = todoItem.Id }, todoItem);
    return CreatedAtAction(nameof(GetTodoItem), new { id = todoItem.Id }, todoItem);
}
The preceding code is an HTTP POST method, as indicated by the [HttpPost] attribute. The method gets the value of the to-do item from the body of the HTTP request.

For more information, see Attribute routing with Http[Verb] attributes.

The CreatedAtAction method:

Returns an HTTP 201 status code if successful. HTTP 201 is the standard response for an HTTP POST method that creates a new resource on the server.
Adds a Location header to the response. The Location header specifies the URI of the newly created to-do item. For more information, see 10.2.2 201 Created.
References the GetTodoItem action to create the Location header's URI. The C# nameof keyword is used to avoid hard-coding the action name in the CreatedAtAction call.

Install http-repl
This tutorial uses http-repl to test the web API.

Run the following command at a command prompt:

.NET CLI

Copy
dotnet tool install -g Microsoft.dotnet-httprepl
If you don't have the .NET 6.0 SDK or runtime installed, install the .NET 6.0 runtime.


Test PostTodoItem
Press Ctrl+F5 to run the app.

Open a new terminal window, and run the following commands. If your app uses a different port number, replace 5001 in the httprepl command with your port number.

.NET CLI

Copy
httprepl https://localhost:5001/api/todoitems
post -h Content-Type=application/json -c "{"name":"walk dog","isComplete":true}"
Here's an example of the output from the command:

Output

Copy
HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8
Date: Tue, 07 Sep 2021 20:39:47 GMT
Location: https://localhost:5001/api/TodoItems/1
Server: Kestrel
Transfer-Encoding: chunked

{
  "id": 1,
  "name": "walk dog",
  "isComplete": true
}
Test the location header URI
To test the location header, copy and paste it into an httprepl get command.

The following example assumes that you're still in an httprepl session. If you ended the previous httprepl session, replace connect with httprepl in the following commands:

.NET CLI

Copy
connect https://localhost:5001/api/todoitems/1
get
Here's an example of the output from the command:

Output

Copy
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Date: Tue, 07 Sep 2021 20:48:10 GMT
Server: Kestrel
Transfer-Encoding: chunked

{
  "id": 1,
  "name": "walk dog",
  "isComplete": true
}
Examine the GET methods
Two GET endpoints are implemented:

GET /api/todoitems
GET /api/todoitems/{id}
You just saw an example of the /api/todoitems/{id} route. Test the /api/todoitems route:

.NET CLI

Copy
connect https://localhost:5001/api/todoitems
get
Here's an example of the output from the command:

Output

Copy
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Date: Tue, 07 Sep 2021 20:59:21 GMT
Server: Kestrel
Transfer-Encoding: chunked

[
  {
    "id": 1,
    "name": "walk dog",
    "isComplete": true
  }
]
This time, the JSON returned is an array of one item.

This app uses an in-memory database. If the app is stopped and started, the preceding GET request will not return any data. If no data is returned, POST data to the app.

Routing and URL paths
The [HttpGet] attribute denotes a method that responds to an HTTP GET request. The URL path for each method is constructed as follows:

Start with the template string in the controller's Route attribute:

C#

Copy
[Route("api/[controller]")]
[ApiController]
public class TodoItemsController : ControllerBase
Replace [controller] with the name of the controller, which by convention is the controller class name minus the "Controller" suffix. For this sample, the controller class name is TodoItemsController, so the controller name is "TodoItems". ASP.NET Core routing is case insensitive.

If the [HttpGet] attribute has a route template (for example, [HttpGet("products")]), append that to the path. This sample doesn't use a template. For more information, see Attribute routing with Http[Verb] attributes.

In the following GetTodoItem method, "{id}" is a placeholder variable for the unique identifier of the to-do item. When GetTodoItem is invoked, the value of "{id}" in the URL is provided to the method in its id parameter.

C#

Copy
[HttpGet("{id}")]
public async Task<ActionResult<TodoItem>> GetTodoItem(long id)
{
    var todoItem = await _context.TodoItems.FindAsync(id);

    if (todoItem == null)
    {
        return NotFound();
    }

    return todoItem;
}
Return values
The return type of the GetTodoItems and GetTodoItem methods is ActionResult<T> type. ASP.NET Core automatically serializes the object to JSON and writes the JSON into the body of the response message. The response code for this return type is 200 OK, assuming there are no unhandled exceptions. Unhandled exceptions are translated into 5xx errors.

ActionResult return types can represent a wide range of HTTP status codes. For example, GetTodoItem can return two different status values:

If no item matches the requested ID, the method returns a 404 status NotFound error code.
Otherwise, the method returns 200 with a JSON response body. Returning item results in an HTTP 200 response.
The PutTodoItem method
Examine the PutTodoItem method:

C#

Copy
[HttpPut("{id}")]
public async Task<IActionResult> PutTodoItem(long id, TodoItem todoItem)
{
    if (id != todoItem.Id)
    {
        return BadRequest();
    }

    _context.Entry(todoItem).State = EntityState.Modified;

    try
    {
        await _context.SaveChangesAsync();
    }
    catch (DbUpdateConcurrencyException)
    {
        if (!TodoItemExists(id))
        {
            return NotFound();
        }
        else
        {
            throw;
        }
    }

    return NoContent();
}
PutTodoItem is similar to PostTodoItem, except it uses HTTP PUT. The response is 204 (No Content). According to the HTTP specification, a PUT request requires the client to send the entire updated entity, not just the changes. To support partial updates, use HTTP PATCH.

If you get an error calling PutTodoItem in the following section, call GET to ensure there's an item in the database.

Test the PutTodoItem method
This sample uses an in-memory database that must be initialized each time the app is started. There must be an item in the database before you make a PUT call. Call GET to ensure there's an item in the database before making a PUT call.

Update the to-do item that has Id = 1 and set its name to "feed fish":

.NET CLI

Copy
connect https://localhost:5001/api/todoitems/1
put -h Content-Type=application/json -c "{"id":1,"name":"feed fish","isComplete":true}"
Here's an example of the output from the command:

Output

Copy
HTTP/1.1 204 No Content
Date: Tue, 07 Sep 2021 21:20:47 GMT
Server: Kestrel
The DeleteTodoItem method
Examine the DeleteTodoItem method:

C#

Copy
[HttpDelete("{id}")]
public async Task<IActionResult> DeleteTodoItem(long id)
{
    var todoItem = await _context.TodoItems.FindAsync(id);
    if (todoItem == null)
    {
        return NotFound();
    }

    _context.TodoItems.Remove(todoItem);
    await _context.SaveChangesAsync();

    return NoContent();
}
Test the DeleteTodoItem method
Delete the to-do item that has Id = 1:

.NET CLI

Copy
connect https://localhost:5001/api/todoitems/1
delete
Here's an example of the output from the command:

Output

Copy
HTTP/1.1 204 No Content
Date: Tue, 07 Sep 2021 21:43:00 GMT
Server: Kestrel

Prevent over-posting
Currently the sample app exposes the entire TodoItem object. Production apps typically limit the data that's input and returned using a subset of the model. There are multiple reasons behind this, and security is a major one. The subset of a model is usually referred to as a Data Transfer Object (DTO), input model, or view model. DTO is used in this tutorial.

A DTO may be used to:

Prevent over-posting.
Hide properties that clients are not supposed to view.
Omit some properties in order to reduce payload size.
Flatten object graphs that contain nested objects. Flattened object graphs can be more convenient for clients.
To demonstrate the DTO approach, update the TodoItem class to include a secret field:

C#

Copy
namespace TodoApi.Models
{
    public class TodoItem
    {
        public long Id { get; set; }
        public string? Name { get; set; }
        public bool IsComplete { get; set; }
        public string? Secret { get; set; }
    }
}
The secret field needs to be hidden from this app, but an administrative app could choose to expose it.

Verify you can post and get the secret field.

Create a DTO model:

C#

Copy
namespace TodoApi.Models
{
    public class TodoItemDTO
    {
        public long Id { get; set; }
        public string? Name { get; set; }
        public bool IsComplete { get; set; }
    }
}
Update the TodoItemsController to use TodoItemDTO:

C#

Copy
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using TodoApi.Models;

namespace TodoApi.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class TodoItemsController : ControllerBase
    {
        private readonly TodoContext _context;

        public TodoItemsController(TodoContext context)
        {
            _context = context;
        }

        // GET: api/TodoItems
        [HttpGet]
        public async Task<ActionResult<IEnumerable<TodoItemDTO>>> GetTodoItems()
        {
            return await _context.TodoItems
                .Select(x => ItemToDTO(x))
                .ToListAsync();
        }

        // GET: api/TodoItems/5
        [HttpGet("{id}")]
        public async Task<ActionResult<TodoItemDTO>> GetTodoItem(long id)
        {
            var todoItem = await _context.TodoItems.FindAsync(id);

            if (todoItem == null)
            {
                return NotFound();
            }

            return ItemToDTO(todoItem);
        }
        // PUT: api/TodoItems/5
        // To protect from overposting attacks, see https://go.microsoft.com/fwlink/?linkid=2123754
        [HttpPut("{id}")]
        public async Task<IActionResult> UpdateTodoItem(long id, TodoItemDTO todoItemDTO)
        {
            if (id != todoItemDTO.Id)
            {
                return BadRequest();
            }

            var todoItem = await _context.TodoItems.FindAsync(id);
            if (todoItem == null)
            {
                return NotFound();
            }

            todoItem.Name = todoItemDTO.Name;
            todoItem.IsComplete = todoItemDTO.IsComplete;

            try
            {
                await _context.SaveChangesAsync();
            }
            catch (DbUpdateConcurrencyException) when (!TodoItemExists(id))
            {
                return NotFound();
            }

            return NoContent();
        }
        // POST: api/TodoItems
        // To protect from overposting attacks, see https://go.microsoft.com/fwlink/?linkid=2123754
        [HttpPost]
        public async Task<ActionResult<TodoItemDTO>> CreateTodoItem(TodoItemDTO todoItemDTO)
        {
            var todoItem = new TodoItem
            {
                IsComplete = todoItemDTO.IsComplete,
                Name = todoItemDTO.Name
            };

            _context.TodoItems.Add(todoItem);
            await _context.SaveChangesAsync();

            return CreatedAtAction(
                nameof(GetTodoItem),
                new { id = todoItem.Id },
                ItemToDTO(todoItem));
        }

        // DELETE: api/TodoItems/5
        [HttpDelete("{id}")]
        public async Task<IActionResult> DeleteTodoItem(long id)
        {
            var todoItem = await _context.TodoItems.FindAsync(id);

            if (todoItem == null)
            {
                return NotFound();
            }

            _context.TodoItems.Remove(todoItem);
            await _context.SaveChangesAsync();

            return NoContent();
        }

        private bool TodoItemExists(long id)
        {
            return _context.TodoItems.Any(e => e.Id == id);
        }

        private static TodoItemDTO ItemToDTO(TodoItem todoItem) =>
            new TodoItemDTO
            {
                Id = todoItem.Id,
                Name = todoItem.Name,
                IsComplete = todoItem.IsComplete
            };
    }
}
Verify you can't post or get the secret field.

Call the web API with JavaScript
See Tutorial: Call an ASP.NET Core web API with JavaScript.

Web API video series
See Video: Beginner's Series to: Web APIs.


Add authentication support to a web API
ASP.NET Core Identity adds user interface (UI) login functionality to ASP.NET Core web apps. To secure web APIs and SPAs, use one of the following:

Azure Active Directory
Azure Active Directory B2C (Azure AD B2C)
Duende Identity Server
Duende Identity Server is an OpenID Connect and OAuth 2.0 framework for ASP.NET Core. Duende Identity Server enables the following security features:

Authentication as a Service (AaaS)
Single sign-on/off (SSO) over multiple application types
Access control for APIs
Federation Gateway
 Important

Duende Software might require you to pay a license fee for production use of Duende Identity Server. For more information, see Migrate from ASP.NET Core 5.0 to 6.0.

For more information, see the Duende Identity Server documentation (Duende Software website).

Publish to Azure
For information on deploying to Azure, see Quickstart: Deploy an ASP.NET web app.

Additional resources
View or download sample code for this tutorial. See how to download.

For more information, see the following resources:

Create web APIs with ASP.NET Core
Tutorial: Create a minimal web API with ASP.NET Core
ASP.NET Core web API documentation with Swagger / OpenAPI
Razor Pages with Entity Framework Core in ASP.NET Core - Tutorial 1 of 8
Routing to controller actions in ASP.NET Core
Controller action return types in ASP.NET Core web API
Deploy ASP.NET Core apps to Azure App Service
Host and deploy ASP.NET Core
Create a web API with ASP.NET Core
Feedback
Submit and view feedback for
