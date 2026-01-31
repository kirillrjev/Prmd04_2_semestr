## Практическая работа №21
## Тема: CRUD-операции с Entity Framework Core
## Вариант: Product

## Модель данных

## Models/Product.cs

namespace Project.Models
{
    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public decimal Price { get; set; }
    }
}

## Контекст базы данных

## Data/AppDbContext.cs

using Microsoft.EntityFrameworkCore;
using Project.Models;

namespace Project.Data
{
    public class AppDbContext : DbContext
    {
        public DbSet<Product> Products { get; set; }

        protected override void OnConfiguring(DbContextOptionsBuilder options)
        {
            options.UseSqlServer(
                "Server=.;Database=ProductsDb;Trusted_Connection=True;TrustServerCertificate=True;"
            );
        }
    }
}

## Реализация CRUD-операций

## Program.cs

using Project.Data;
using Project.Models;

using var context = new AppDbContext();

// CREATE
var product = new Product
{
    Name = "Laptop",
    Price = 90000
};
context.Products.Add(product);
context.SaveChanges();
Console.WriteLine("Продукт добавлен");

// READ (все записи)
var products = context.Products.AsNoTracking().ToList();
Console.WriteLine("Список продуктов:");
foreach (var p in products)
{
    Console.WriteLine($"{p.Id}. {p.Name} - {p.Price} руб.");
}

// READ (одна запись)
var singleProduct = context.Products.FirstOrDefault(p => p.Id == product.Id);

// UPDATE
if (singleProduct != null)
{
    singleProduct.Price = 85000;
    context.Products.Update(singleProduct);
    context.SaveChanges();
    Console.WriteLine("Цена обновлена");
}

// DELETE
context.Products.Remove(singleProduct);
context.SaveChanges();
Console.WriteLine("Продукт удалён");

