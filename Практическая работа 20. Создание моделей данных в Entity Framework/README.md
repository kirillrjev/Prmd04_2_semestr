## Практическая работа №20
## Тема: Создание моделей данных в Entity Framework Core
## Вариант: Product

## Модель данных (Data Annotations)

## Models/Product.cs

using System.ComponentModel.DataAnnotations;

namespace Project.Models
{
    public class Product
    {
        [Key]
        public int Id { get; set; }

        [Required]
        [MaxLength(100)]
        public string Name { get; set; }

        [Range(0, 1_000_000)]
        public decimal Price { get; set; }
    }
}

## десь используются Data Annotations:

[Key] — первичный ключ

[Required] — обязательное поле

[MaxLength] — ограничение длины

[Range] — допустимый диапазон значений

## Контекст базы данных (Fluent API)

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

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<Product>()
                .Property(p => p.Price)
                .HasColumnType("decimal(18,2)");

            modelBuilder.Entity<Product>()
                .HasIndex(p => p.Name)
                .IsUnique(false);
        }
    }
}

## десь применяется Fluent API:

## настройка типа decimal(18,2)

## индекс по полю Name

## CRUD-операции

## Program.cs

using Project.Data;
using Project.Models;

using var context = new AppDbContext();

// CREATE
var product = new Product
{
    Name = "Laptop",
    Price = 85000
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
product.Price = 80000;
context.SaveChanges();
Console.WriteLine("Цена обновлена");

// DELETE
context.Products.Remove(product);
context.SaveChanges();
Console.WriteLine("Продукт удалён");



