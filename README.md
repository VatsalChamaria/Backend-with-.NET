**Program.cs**

using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

using System.ComponentModel.DataAnnotations;

public class User
{
    public int Id { get; set; }

    [Required(ErrorMessage = "Name is required")]
    [StringLength(50, ErrorMessage = "Name must be less than 50 characters")]
    public string Name { get; set; }

    [Required(ErrorMessage = "Email is required")]
    [EmailAddress(ErrorMessage = "Invalid email format")]
    public string Email { get; set; }
}

var builder = WebApplication.CreateBuilder(args);

// Add services to the container
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();
app.Run();

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddLogging();
var app = builder.Build();

app.UseMiddleware<LoggingMiddleware>(); // Add logging middleware

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();
app.Run();

builder.Services.AddAuthentication("Bearer")
    .AddJwtBearer(options =>
    {
        options.Authority = "https://your-auth-server.com";
        options.Audience = "UserManagementAPI";
    });

app.UseAuthentication();
app.UseAuthorization();

if (!ModelState.IsValid)
{
    var errors = ModelState.Values.SelectMany(v => v.Errors).Select(e => e.ErrorMessage);
    return BadRequest(new { Message = "Validation Failed", Errors = errors });
}

**Http request**
[HttpPost]
public ActionResult<User> CreateUser([FromBody] User user)
{
    if (!ModelState.IsValid)
        return BadRequest(ModelState);

    user.Id = users.Count + 1;
    users.Add(user);
    return CreatedAtAction(nameof(GetUserById), new { id = user.Id }, user);
}

[HttpPut("{id}")]
public IActionResult UpdateUser(int id, [FromBody] User updatedUser)
{
    if (!ModelState.IsValid)
        return BadRequest(ModelState);

    var user = users.FirstOrDefault(u => u.Id == id);
    if (user == null)
        return NotFound(new { Message = "User not found" });

    user.Name = updatedUser.Name;
    user.Email = updatedUser.Email;
    return Ok(new { Message = "User updated successfully" });
}

using Microsoft.AspNetCore.Authorization;

[Authorize] // Requires authentication for all API requests
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase

**LoggingMiddleware.cs**
using Microsoft.AspNetCore.Http;
using System.Diagnostics;
using System.Threading.Tasks;
using Microsoft.Extensions.Logging;

public class LoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<LoggingMiddleware> _logger;

    public LoggingMiddleware(RequestDelegate next, ILogger<LoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var stopwatch = Stopwatch.StartNew();
        _logger.LogInformation($"Incoming Request: {context.Request.Method} {context.Request.Path}");

        await _next(context);

        stopwatch.Stop();
        _logger.LogInformation($"Response Status: {context.Response.StatusCode} - Processed in {stopwatch.ElapsedMilliseconds} ms");
    }
}
