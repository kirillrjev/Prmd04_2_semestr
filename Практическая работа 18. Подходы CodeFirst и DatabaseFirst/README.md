## Практическая работа №18
## Тема: Подходы CodeFirst и DatabaseFirst

## Вариант: Создание базы данных продуктов (CodeFirst)

## Модель данных (CodeFirst)

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

## CRUD-операции (консольное приложение)

## Program.cs

using Project.Data;
using Project.Models;

using var context = new AppDbContext();

// CREATE
var product = new Product
{
    Name = "Laptop",
    Price = 75000
};
context.Products.Add(product);
context.SaveChanges();
Console.WriteLine("Продукт добавлен");

// READ
var products = context.Products.ToList();
Console.WriteLine("Список продуктов:");
foreach (var p in products)
{
    Console.WriteLine($"{p.Id}. {p.Name} - {p.Price} руб.");
}

// UPDATE
product.Price = 70000;
context.SaveChanges();
Console.WriteLine("Цена обновлена");

// DELETE
context.Products.Remove(product);
context.SaveChanges();
Console.WriteLine("Продукт удалён");

## Создание базы данных (миграции)

## В Package Manager Console или терминале:


