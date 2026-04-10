# Практическая работа №33: Логгирование. Serilog

## Вариант 1

### Задание
Настроить Serilog для записи логов в:
- консоль
- файл

Реализовать логирование:
- информационных событий
- предупреждений
- ошибок

---

## Теоретическая часть

### Логгирование
Логгирование — это процесс записи информации о работе приложения.

Уровни:
- Information — обычные события
- Warning — предупреждения
- Error — ошибки

---

### Serilog
Serilog — библиотека структурированного логирования для .NET.

Поддерживает:
- консоль
- файлы
- базы данных

---

## Практический пример

---

## ⚙️ Настройка Serilog (Program.cs)

```csharp id="serilog_program"
using Serilog;

var builder = WebApplication.CreateBuilder(args);

// настройка логгера
Log.Logger = new LoggerConfiguration()
    .WriteTo.Console()
    .WriteTo.File("Logs/log-.txt", rollingInterval: RollingInterval.Day)
    .MinimumLevel.Information()
    .CreateLogger();

// подключение Serilog
builder.Host.UseSerilog();

builder.Services.AddControllers();

var app = builder.Build();

// логирование HTTP запросов
app.UseSerilogRequestLogging();

app.MapControllers();

app.Run();

using Microsoft.AspNetCore.Mvc;
using Serilog;

[ApiController]
[Route("api/[controller]")]
public class SampleController : ControllerBase
{
    [HttpGet("test")]
    public IActionResult Test()
    {
        Log.Information("Тестовый запрос выполнен в {Time}", DateTime.Now);

        try
        {
            throw new Exception("Тестовая ошибка");
        }
        catch (Exception ex)
        {
            Log.Error(ex, "Произошла ошибка при выполнении запроса");
        }

        return Ok("Логи записаны");
    }

    [HttpGet("user")]
    public IActionResult UserAction()
    {
        string user = "admin";

        Log.Information("Пользователь {User} вошел в систему", user);
        Log.Warning("Попытка доступа к защищенному ресурсу");

        return Ok("Действие пользователя залогировано");
    }
}