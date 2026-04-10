# Практическая работа №32: Работа с файлами в ASP.NET Core

## Вариант 1

### Задание
Реализовать загрузку файлов через Razor Page:
- загрузка одного файла
- проверка типа и размера файла
- сохранение с уникальным именем
- вывод сообщения о результате

---

## Теоретическая часть

### Работа с файлами
В ASP.NET Core для загрузки файлов используется интерфейс:

- `IFormFile` — один файл
- `FileStream` — запись на диск
- `Path` — работа с путями

---

## Практический пример

---

## 📄 Страница загрузки (Pages/Upload.cshtml)

```html id="upload32_view"
@page
@model UploadModel

<h2>Загрузка файла</h2>

<form method="post" enctype="multipart/form-data">
    <input type="file" asp-for="File" />
    <br><br>
    <button type="submit">Загрузить</button>
</form>

@if (Model.Message != null)
{
    <p>@Model.Message</p>
}

using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;

public class UploadModel : PageModel
{
    [BindProperty]
    public IFormFile File { get; set; }

    public string Message { get; set; }

    public async Task<IActionResult> OnPostAsync()
    {
        if (File == null || File.Length == 0)
        {
            Message = "Выберите файл для загрузки!";
            return Page();
        }

        // Проверка размера (5 MB)
        if (File.Length > 5 * 1024 * 1024)
        {
            Message = "Файл слишком большой (макс 5 MB)";
            return Page();
        }

        // Проверка типа файла
        var allowedTypes = new[] { "image/jpeg", "image/png", "application/pdf" };

        if (!allowedTypes.Contains(File.ContentType))
        {
            Message = "Недопустимый тип файла";
            return Page();
        }

        // Папка для загрузки
        var uploadsFolder = Path.Combine(Directory.GetCurrentDirectory(), "Uploads");

        if (!Directory.Exists(uploadsFolder))
        {
            Directory.CreateDirectory(uploadsFolder);
        }

        // Уникальное имя файла
        var fileName = Path.GetRandomFileName() + Path.GetExtension(File.FileName);
        var filePath = Path.Combine(uploadsFolder, fileName);

        // Сохранение файла
        using (var stream = new FileStream(filePath, FileMode.Create))
        {
            await File.CopyToAsync(stream);
        }

        Message = "Файл успешно загружен!";
        return Page();
    }
}

