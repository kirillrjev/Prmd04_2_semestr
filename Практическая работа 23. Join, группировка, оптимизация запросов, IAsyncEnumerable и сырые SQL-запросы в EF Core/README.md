## Практическая работа №23
## Тема: Join, группировка, оптимизация запросов, IAsyncEnumerable и сырые SQL-запросы в EF Core
## Вариант: Customer + Order

## Модели данных
Customer

Models/Customer.cs

namespace Project.Models
{
    public class Customer
    {
        public int Id { get; set; }
        public string FullName { get; set; }
        public ICollection<Order> Orders { get; set; }
    }
}

## Order

Models/Order.cs

namespace Project.Models
{
    public class Order
    {
        public int Id { get; set; }
        public int CustomerId { get; set; }
        public Customer Customer { get; set; }
        public decimal TotalAmount { get; set; }
        public DateTime OrderDate { get; set; }
    }
}

## Product (для сырого SQL)

Models/Product.cs

namespace Project.Models
{
    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public decimal Price { get; set; }
    }
}

## DbContext

Data/ShopContext.cs

using Microsoft.EntityFrameworkCore;
using Project.Models;

namespace Project.Data
{
    public class ShopContext : DbContext
    {
        public DbSet<Customer> Customers { get; set; }
        public DbSet<Order> Orders { get; set; }
        public DbSet<Product> Products { get; set; }

        protected override void OnConfiguring(DbContextOptionsBuilder options)
        {
            options.UseSqlServer(
                "Server=.;Database=ShopDb;Trusted_Connection=True;TrustServerCertificate=True;"
            );
        }
    }
}

## Join + группировка и агрегация

Program.cs

using Microsoft.EntityFrameworkCore;
using Project.Data;

using var context = new ShopContext();

// JOIN + GROUP BY
var customerOrders =
    from order in context.Orders
    join customer in context.Customers
        on order.CustomerId equals customer.Id
    group order by customer.FullName into g
    select new
    {
        CustomerName = g.Key,
        TotalOrders = g.Count(),
        TotalAmount = g.Sum(o => o.TotalAmount)
    };

Console.WriteLine("Сумма заказов по клиентам:");
foreach (var item in customerOrders)
{
    Console.WriteLine($"{item.CustomerName}: заказов = {item.TotalOrders}, сумма = {item.TotalAmount}");
}

## Оптимизация запросов
var customersWithOrders = context.Customers
    .Include(c => c.Orders)
    .Where(c => c.Orders.Any())
    .Select(c => new
    {
        c.FullName,
        OrdersCount = c.Orders.Count
    })
    .AsNoTracking()
    .ToList();

Console.WriteLine("Клиенты с заказами:");
foreach (var c in customersWithOrders)
{
    Console.WriteLine($"{c.FullName} - {c.OrdersCount} заказов");
}

## Использовано:

Include — eager loading

Where — фильтрация

Select — выбор нужных колонок

AsNoTracking — оптимизация чтения

## IAsyncEnumerable (асинхронный перебор)

Console.WriteLine("Асинхронный вывод заказов:");

await foreach (var order in context.Orders.AsAsyncEnumerable())
{
    Console.WriteLine($"{order.Id} | {order.OrderDate} | {order.TotalAmount}");
}

## Сырые SQL-запросы
SELECT (FromSqlRaw)

var expensiveProducts = context.Products
    .FromSqlRaw("SELECT * FROM Products WHERE Price > {0}", 50000)
    .AsNoTracking()
    .ToList();

Console.WriteLine("Дорогие продукты:");
foreach (var p in expensiveProducts)
{
    Console.WriteLine($"{p.Name} - {p.Price}");
}

## Команда (ExecuteSqlRaw) — при необходимости

context.Database.ExecuteSqlRaw(
    "UPDATE Products SET Price = Price * 0.9 WHERE Price > {0}", 100000
);
