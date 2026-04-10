# Практическая работа №28: Разработка приложения Blazor

## Вариант 1

### Задание
Создать компонент для добавления и отображения списка продуктов с полями:
- название
- цена

---

## Теоретическая часть

### Blazor
Blazor — это технология Microsoft для создания интерактивных веб-приложений на C#.

Типы:
- **Blazor Server** — выполняется на сервере
- **Blazor WebAssembly** — выполняется в браузере

Особенности:
- компоненты `.razor`
- двусторонняя привязка данных `@bind`
- обработка событий `@onclick`
- отсутствие необходимости в JavaScript для логики UI

---

## Практический пример

---

### 📦 Модель (Models/Product.cs)

```csharp
public class Product
{
    public string Name { get; set; }
    public decimal Price { get; set; }
}


@page "/products"

<h3>Список продуктов</h3>

<ul>
    @foreach (var p in products)
    {
        <li>@p.Name - @p.Price ₽</li>
    }
</ul>

<h3>Добавить продукт</h3>

<input placeholder="Название" @bind="newProduct.Name" />
<br />
<input type="number" placeholder="Цена" @bind="newProduct.Price" />
<br />
<button @onclick="AddProduct">Добавить</button>

@code {
    private List<Product> products = new();

    private Product newProduct = new();

    private void AddProduct()
    {
        if (!string.IsNullOrWhiteSpace(newProduct.Name))
        {
            products.Add(new Product
            {
                Name = newProduct.Name,
                Price = newProduct.Price
            });

            newProduct = new Product();
        }
    }
}