# Практическая работа №26: Разработка приложения Razor Page

## Вариант 1

### Задание
Создать страницу с формой ввода имени пользователя. После отправки формы вывести приветствие с именем пользователя.

---

## Пример кода

### Модель

```csharp
public class UserModel
{
    public string Name { get; set; }
}
@page
@model HelloPageModel

<h2>Ввод имени пользователя</h2>

<form method="post">
    <label>Имя:</label>
    <input type="text" asp-for="User.Name" />

    <button type="submit">Отправить</button>
</form>

@if (Model.Submitted)
{
    <p>Привет, @Model.User.Name!</p>
}  

using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;

public class HelloPageModel : PageModel
{
    [BindProperty]
    public UserModel User { get; set; } = new UserModel();

    public bool Submitted { get; set; }

    public void OnGet()
    {
    }

    public void OnPost()
    {
        if (ModelState.IsValid)
        {
            Submitted = true;
        }
    }
}