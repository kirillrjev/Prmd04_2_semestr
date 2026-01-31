## Самостоятельная работа №5
Разработка системы учёта заказов (EF Core, Code First)

Тип проекта: Console App (.NET 6/7)
БД: SQL Server (LocalDB)

## Модели данных
Customer.cs
public class Customer
{
    public int Id { get; set; }
    public string FullName { get; set; } = null!;
    public string Email { get; set; } = null!;
    public string Phone { get; set; } = null!;

    public ICollection<Order> Orders { get; set; } = new List<Order>();
}

## Product.cs
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = null!;
    public decimal Price { get; set; }
    public int Stock { get; set; }

    public ICollection<OrderItem> OrderItems { get; set; } = new List<OrderItem>();
}

## Order.cs
public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public Customer Customer { get; set; } = null!;

    public DateTime OrderDate { get; set; }
    public decimal TotalAmount { get; set; }

    public ICollection<OrderItem> OrderItems { get; set; } = new List<OrderItem>();
}

OrderItem.cs
public class OrderItem
{
    public int Id { get; set; }

    public int OrderId { get; set; }
    public Order Order { get; set; } = null!;

    public int ProductId { get; set; }
    public Product Product { get; set; } = null!;

    public int Quantity { get; set; }
    public decimal Price { get; set; }
}

## DbContext (Code First)
ShopContext.cs
using Microsoft.EntityFrameworkCore;

public class ShopContext : DbContext
{
    public DbSet<Customer> Customers => Set<Customer>();
    public DbSet<Product> Products => Set<Product>();
    public DbSet<Order> Orders => Set<Order>();
    public DbSet<OrderItem> OrderItems => Set<OrderItem>();

    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        options.UseSqlServer(
            "Server=(localdb)\\mssqllocaldb;Database=ShopDb_SR5;Trusted_Connection=True;"
        );
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<OrderItem>()
            .HasOne(oi => oi.Order)
            .WithMany(o => o.OrderItems)
            .HasForeignKey(oi => oi.OrderId);

        modelBuilder.Entity<OrderItem>()
            .HasOne(oi => oi.Product)
            .WithMany(p => p.OrderItems)
            .HasForeignKey(oi => oi.ProductId);
    }
}

## Основная логика (CRUD + транзакция)
Program.cs
using Microsoft.EntityFrameworkCore;

using var context = new ShopContext();
context.Database.EnsureDeleted();
context.Database.EnsureCreated();

// CREATE: Клиент
var customer = new Customer
{
    FullName = "John Doe",
    Email = "john@example.com",
    Phone = "1234567890"
};
context.Customers.Add(customer);

// CREATE: Товары
var laptop = new Product { Name = "Laptop", Price = 1500, Stock = 10 };
var mouse = new Product { Name = "Mouse", Price = 50, Stock = 100 };
context.Products.AddRange(laptop, mouse);

context.SaveChanges();

Console.WriteLine("Данные добавлены.");

// CREATE ORDER + TRANSACTION
using var transaction = await context.Database.BeginTransactionAsync();
try
{
    var order = new Order
    {
        CustomerId = customer.Id,
        OrderDate = DateTime.Now
    };

    var item1 = new OrderItem
    {
        ProductId = laptop.Id,
        Quantity = 1,
        Price = laptop.Price
    };

    var item2 = new OrderItem
    {
        ProductId = mouse.Id,
        Quantity = 2,
        Price = mouse.Price
    };

    order.OrderItems.Add(item1);
    order.OrderItems.Add(item2);

    order.TotalAmount =
        item1.Price * item1.Quantity +
        item2.Price * item2.Quantity;

    laptop.Stock -= item1.Quantity;
    mouse.Stock -= item2.Quantity;

    context.Orders.Add(order);
    await context.SaveChangesAsync();
    await transaction.CommitAsync();

    Console.WriteLine("Заказ успешно создан.");
}
catch
{
    await transaction.RollbackAsync();
    Console.WriteLine("Ошибка при создании заказа. Операция отменена.");
}

// READ: Просмотр заказов с деталями
var orders = context.Orders
    .Include(o => o.Customer)
    .Include(o => o.OrderItems)
        .ThenInclude(oi => oi.Product)
    .AsNoTracking()
    .ToList();

Console.WriteLine("\nСписок заказов:");
foreach (var o in orders)
{
    Console.WriteLine($"Заказ #{o.Id} | Клиент: {o.Customer.FullName} | Сумма: {o.TotalAmount}");
    foreach (var item in o.OrderItems)
    {
        Console.WriteLine($"  - {item.Product.Name} x{item.Quantity} = {item.Price * item.Quantity}");
    }
}
