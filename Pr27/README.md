# Практическая работа №27: Разработка приложения MVC

## Вариант 1

### Задание
Создать приложение MVC для списка продуктов. Реализовать:
- отображение списка продуктов
- добавление нового продукта
- удаление продукта

---

## Теоретическая часть

### MVC в ASP.NET Core
MVC (Model-View-Controller) — архитектура, разделяющая приложение на:

- **Model** — данные
- **View** — отображение
- **Controller** — логика обработки запросов

---

## Практический пример

---

###  Модель (Models/Product.cs)

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}


using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;
using System.Linq;

public class ProductsController : Controller
{
    private static List<Product> products = new List<Product>();

    public IActionResult Index()
    {
        return View(products);
    }

    public IActionResult Create()
    {
        return View();
    }

    [HttpPost]
    public IActionResult Create(Product product)
    {
        product.Id = products.Count + 1;
        products.Add(product);
        return RedirectToAction("Index");
    }

    public IActionResult Delete(int id)
    {
        var product = products.FirstOrDefault(p => p.Id == id);
        if (product != null)
        {
            products.Remove(product);
        }
        return RedirectToAction("Index");
    }
}

@model List<Product>

<h2>Список продуктов</h2>

<table border="1">
    <tr>
        <th>Id</th>
        <th>Название</th>
        <th>Цена</th>
        <th>Действия</th>
    </tr>

@foreach (var item in Model)
{
    <tr>
        <td>@item.Id</td>
        <td>@item.Name</td>
        <td>@item.Price</td>
        <td>
            <a asp-action="Delete" asp-route-id="@item.Id">Удалить</a>
        </td>
    </tr>
}
</table>

<br>

<a asp-action="Create">Добавить продукт</a> 

@model Product

<h2>Добавление продукта</h2>

<form asp-action="Create" method="post">
    <label>Название:</label>
    <input asp-for="Name" />

    <br>

    <label>Цена:</label>
    <input asp-for="Price" />

    <br><br>

    <button type="submit">Добавить</button>
</form>