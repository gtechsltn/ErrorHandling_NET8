# Error Handling in .NET 8.0

Mastering Error Handling in .NET Core: Best Practices and Real-World Examples

https://medium.com/@michaelmaurice410/mastering-error-handling-in-net-core-best-practices-and-real-world-examples-fbd7b327a7bc

## 1. Global Error Handling Middleware
```
public class ErrorHandlerMiddleware
{
    private readonly RequestDelegate _next;
    public ErrorHandlerMiddleware(RequestDelegate next)
    {
        _next = next;
    }
    public async Task Invoke(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            await HandleExceptionAsync(context, ex);
        }
    }
    private static Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        var response = context.Response;
        response.ContentType = "application/json";
        var errorResponse = new
        {
            message = exception.Message,
            stackTrace = exception.StackTrace,
            innerException = exception.InnerException?.Message
        };
        return context.Response.WriteAsJsonAsync(errorResponse);
    }
}
```

### Program.cs
```
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseMiddleware<ErrorHandlerMiddleware>();
    // Other middleware
}
```

## 2. Custom Exception Filter
```
public class CustomExceptionFilter : IExceptionFilter
{
    public void OnException(ExceptionContext context)
    {
        var exception = context.Exception;
        var result = new ObjectResult(new { error = exception.Message })
        {
            StatusCode = (int)HttpStatusCode.InternalServerError
        };
context.Result = result;
    }
}
```

### Program.cs
```
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllersWithViews(options =>
    {
        options.Filters.Add<CustomExceptionFilter>();
    });
}
```

## 3. Try-Catch Blocks
```
public async Task<IActionResult> GetDataAsync(int id)
{
    try
    {
        var data = await _dataService.GetDataAsync(id);
        return Ok(data);
    }
    catch (DataNotFoundException ex)
    {
        return NotFound(ex.Message);
    }
    catch (Exception ex)
    {
        return StatusCode(500, "An error occurred while processing your request.");
    }
}
```

## Advanced Error Handling Techniques Using Polly for Resilience

### Using FluentValidation for Input Validation
```
public class UserModelValidator : AbstractValidator<UserModel>
{
    public UserModelValidator()
    {
        RuleFor(user => user.Email).NotEmpty().EmailAddress();
        RuleFor(user => user.Password).NotEmpty().MinimumLength(8);
    }
}
```

### Polly Retry Policy
```
var retryPolicy = Policy
    .Handle<HttpRequestException>()
    .WaitAndRetryAsync(3, retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)));
await retryPolicy.ExecuteAsync(() => _httpClient.GetAsync("https://api.example.com/data"));
```
