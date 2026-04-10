# Практическая работа №31: Валидация. FluentValidation

## Вариант 1

### Задание
Реализовать валидацию модели пользователя (имя, email, пароль) с использованием FluentValidation.

---

## Теоретическая часть

### FluentValidation
FluentValidation — библиотека для удобной и читаемой валидации моделей в .NET.

Преимущества:
- читаемый код правил
- переиспользуемые валидаторы
- интеграция с ASP.NET Core

---

## Практический пример

---

## 📦 Модель (Models/User.cs)

```csharp
public class User
{
    public string Username { get; set; }
    public string Email { get; set; }
    public string Password { get; set; }
}

using FluentValidation;

public class UserValidator : AbstractValidator<User>
{
    public UserValidator()
    {
        RuleFor(u => u.Username)
            .NotEmpty()
            .WithMessage("Имя пользователя не должно быть пустым");

        RuleFor(u => u.Email)
            .NotEmpty()
            .EmailAddress()
            .WithMessage("Неверный формат email");

        RuleFor(u => u.Password)
            .MinimumLength(6)
            .WithMessage("Пароль должен быть не менее 6 символов");
    }
}


using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    [HttpPost("register")]
    public IActionResult Register(User user)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }

        return Ok("Пользователь успешно зарегистрирован");
    }
}

using FluentValidation;
using FluentValidation.AspNetCore;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

// подключение FluentValidation
builder.Services.AddFluentValidationAutoValidation();
builder.Services.AddValidatorsFromAssemblyContaining<UserValidator>();

var app = builder.Build();

app.MapControllers();

app.Run();

