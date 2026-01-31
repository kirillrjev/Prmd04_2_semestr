## Практическая работа №22
## Тема: Миграции и управление схемой базы данных в EF Core
## Вариант: Product

## Модель данных (начальная версия)

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

## Создание первой миграции

Команды в терминале проекта:

dotnet ef migrations add InitialCreate
dotnet ef database update


## В базе данных создаётся таблица Products с колонками:

Id

Name

Price

## Изменение модели (добавление нового поля)

Models/Product.cs (обновлённая версия)

namespace Project.Models
{
    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public decimal Price { get; set; }
        public string Description { get; set; } // новое поле
    }
}

## Создание второй миграции
dotnet ef migrations add AddDescriptionToProduct
dotnet ef database update


## В таблицу Products добавляется колонка Description.

7. Откат миграции (при необходимости)
dotnet ef database update InitialCreate
dotnet ef migrations remove

## Пример использования обновлённой модели

Program.cs

using Project.Data;
using Project.Models;

using var context = new AppDbContext();

var product = new Product
{
    Name = "Laptop",
    Price = 90000,
    Description = "Игровой ноутбук"
};

context.Products.Add(product);
context.SaveChanges();

Console.WriteLine("Продукт с описанием добавлен");

