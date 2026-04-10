# Практическая работа №30: Фоновые задачи. Hangfire

## Вариант 1

### Задание
Реализовать фоновые задачи с использованием Hangfire:
- Fire-and-forget
- Delayed
- Recurring

Настроить Dashboard для мониторинга выполнения задач.

---

## Теоретическая часть

### Фоновые задачи
Фоновые задачи выполняются независимо от основного потока приложения.

Примеры:
- отправка email
- генерация отчетов
- очистка данных

---

### Hangfire
Hangfire — библиотека для выполнения фоновых задач в .NET.

Возможности:
- очередь задач
- отложенные задачи
- повторяющиеся задачи (CRON)
- Dashboard для мониторинга

---

## Практический пример

---

## ⚙️ Настройка Hangfire (Program.cs)

```csharp id="hangfire_program"
using Hangfire;
using Hangfire.MemoryStorage;

var builder = WebApplication.CreateBuilder(args);

// сервисы
builder.Services.AddControllers();

// Hangfire
builder.Services.AddHangfire(config =>
    config.UseSimpleAssemblyNameTypeSerializer()
          .UseRecommendedSerializerSettings()
          .UseMemoryStorage()
);

builder.Services.AddHangfireServer();

// сервис задач
builder.Services.AddSingleton<EmailService>();

var app = builder.Build();

// Dashboard Hangfire
app.UseHangfireDashboard("/hangfire");

app.MapControllers();

app.Run();

public class EmailService
{
    public void SendEmail(string to, string subject)
    {
        Console.WriteLine($"Email sent to {to} | Subject: {subject}");
    }
}

using Hangfire;
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
public class TasksController : ControllerBase
{
    private readonly EmailService _emailService;

    public TasksController(EmailService emailService)
    {
        _emailService = emailService;
    }

    // Fire-and-forget
    [HttpPost("fire-and-forget")]
    public IActionResult FireAndForget()
    {
        BackgroundJob.Enqueue(() =>
            _emailService.SendEmail("user@example.com", "Fire-and-Forget Task"));

        return Ok("Fire-and-forget task queued");
    }

    // Delayed
    [HttpPost("delayed")]
    public IActionResult Delayed()
    {
        BackgroundJob.Schedule(() =>
            _emailService.SendEmail("user@example.com", "Delayed Task"),
            TimeSpan.FromMinutes(1));

        return Ok("Delayed task scheduled");
    }

    // Recurring
    [HttpPost("recurring")]
    public IActionResult Recurring()
    {
        RecurringJob.AddOrUpdate(
            "daily-email",
            () => _emailService.SendEmail("user@example.com", "Daily Email"),
            Cron.Daily);

        return Ok("Recurring task scheduled");
    }
}

