# Практическая работа №29: Работа с JWT токенами, ролями и разрешениями

## Вариант 1

### Задание
Реализовать регистрацию и логин пользователей с JWT.
Создать защищённый эндпоинт `/api/products`, доступный только для роли `Admin`.

---

## Теоретическая часть

### JWT (JSON Web Token)
JWT — это токен для безопасной передачи данных между клиентом и сервером.

Состоит из:
- Header (алгоритм)
- Payload (данные пользователя)
- Signature (подпись)

---

### Роли
- **Admin** — полный доступ
- **User** — ограниченный доступ

---

## Практический пример

---

## 📦 Модель пользователя (Models/User.cs)

```csharp
public class User
{
    public string Username { get; set; }
    public string Password { get; set; } // учебный пример
    public string Role { get; set; }
}

using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using Microsoft.IdentityModel.Tokens;
using System.Text;

public class JwtService
{
    private readonly string _secret;

    public JwtService(string secret)
    {
        _secret = secret;
    }

    public string GenerateToken(User user)
    {
        var claims = new[]
        {
            new Claim(ClaimTypes.Name, user.Username),
            new Claim(ClaimTypes.Role, user.Role)
        };

        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_secret));
        var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: "MyApp",
            audience: "MyApp",
            claims: claims,
            expires: DateTime.UtcNow.AddHours(1),
            signingCredentials: creds
        );

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}

using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;
using System.Linq;

[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly JwtService _jwtService;

    private static List<User> users = new();

    public AuthController(JwtService jwtService)
    {
        _jwtService = jwtService;
    }

    [HttpPost("register")]
    public IActionResult Register(User user)
    {
        users.Add(user);
        return Ok("User registered");
    }

    [HttpPost("login")]
    public IActionResult Login(User login)
    {
        var user = users.FirstOrDefault(u =>
            u.Username == login.Username &&
            u.Password == login.Password);

        if (user == null)
            return Unauthorized();

        var token = _jwtService.GenerateToken(user);

        return Ok(new { token });
    }
}

using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Authorization;

[ApiController]
[Route("api/[controller]")]
[Authorize(Roles = "Admin")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult Get()
    {
        return Ok(new[] { "Product1", "Product2" });
    }
}   

using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using System.Text;

var builder = WebApplication.CreateBuilder(args);

var key = "SUPER_SECRET_KEY_12345";

builder.Services.AddSingleton(new JwtService(key));

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = "MyApp",
            ValidAudience = "MyApp",
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(key))
        };
    });

builder.Services.AddAuthorization();
builder.Services.AddControllers();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.Run();