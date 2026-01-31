## Практическая работа №25
Уровни изоляции и стратегии конкурентного доступа в EF Core

Тип проекта: Console App (.NET 6/7)
БД: SQL Server (важно для UPDLOCK)

## Модель с поддержкой оптимистической блокировки
Product.cs
using System.ComponentModel.DataAnnotations;

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = null!;
    public decimal Price { get; set; }

    [Timestamp]
    public byte[] RowVersion { get; set; } = null!;
}

## DbContext
ShopContext.cs
using Microsoft.EntityFrameworkCore;

public class ShopContext : DbContext
{
    public DbSet<Product> Products => Set<Product>();

    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        options.UseSqlServer(
            "Server=(localdb)\\mssqllocaldb;Database=ShopDb25;Trusted_Connection=True;"
        );
    }
}

## Демонстрация конкурентного доступа
Program.cs
using Microsoft.EntityFrameworkCore;
using System.Data;

using var context = new ShopContext();
context.Database.EnsureDeleted();
context.Database.EnsureCreated();

// Начальные данные
context.Products.Add(new Product
{
    Name = "Laptop",
    Price = 1500
});
context.SaveChanges();

Console.WriteLine("=== ОПТИМИСТИЧЕСКАЯ БЛОКИРОВКА ===");

var context1 = new ShopContext();
var context2 = new ShopContext();

var product1 = await context1.Products.FirstAsync();
var product2 = await context2.Products.FirstAsync();

product1.Price = 1600;
await context1.SaveChangesAsync();

try
{
    product2.Price = 1700;
    await context2.SaveChangesAsync();
}
catch (DbUpdateConcurrencyException)
{
    Console.WriteLine("Конфликт обновления! Данные были изменены другим пользователем.");
}

Console.WriteLine();
Console.WriteLine("=== ПЕССИМИСТИЧЕСКАЯ БЛОКИРОВКА ===");

using var pessimisticContext = new ShopContext();
using var transaction = await pessimisticContext.Database
    .BeginTransactionAsync(IsolationLevel.ReadCommitted);

var lockedProduct = await pessimisticContext.Products
    .FromSqlRaw(
        "SELECT * FROM Products WITH (UPDLOCK) WHERE Id = {0}", 1)
    .FirstAsync();

lockedProduct.Price = 1800;
await pessimisticContext.SaveChangesAsync();
await transaction.CommitAsync();

Console.WriteLine("Обновление выполнено с пессимистической блокировкой.");
