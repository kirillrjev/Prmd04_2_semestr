## Практическая работа №24

## Тема: Транзакции и паттерн Unit of Work в EF Core
## Тип проекта: Console App (.NET 6/7)

## Модели
Product.cs
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = null!;
    public decimal Price { get; set; }
}

## Order.cs

public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public decimal TotalAmount { get; set; }
}

## DbContext
ShopContext.cs

using Microsoft.EntityFrameworkCore;

public class ShopContext : DbContext
{
    public DbSet<Product> Products => Set<Product>();
    public DbSet<Order> Orders => Set<Order>();

    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        options.UseSqlite("Data Source=shop.db");
    }
}

## епозитории
Интерфейсы
IProductRepository.cs

public interface IProductRepository
{
    void Add(Product product);
    IEnumerable<Product> GetAll();
}

## IOrderRepository.cs
public interface IOrderRepository
{
    void Add(Order order);
    IEnumerable<Order> GetAll();
}

## Реализация
ProductRepository.cs
public class ProductRepository : IProductRepository
{
    private readonly ShopContext _context;

    public ProductRepository(ShopContext context)
    {
        _context = context;
    }

    public void Add(Product product)
    {
        _context.Products.Add(product);
    }

    public IEnumerable<Product> GetAll()
    {
        return _context.Products.ToList();
    }
}

## OrderRepository.cs
public class OrderRepository : IOrderRepository
{
    private readonly ShopContext _context;

    public OrderRepository(ShopContext context)
    {
        _context = context;
    }

    public void Add(Order order)
    {
        _context.Orders.Add(order);
    }

    public IEnumerable<Order> GetAll()
    {
        return _context.Orders.ToList();
    }
}

## Unit of Work
IUnitOfWork.cs
public interface IUnitOfWork : IDisposable
{
    IProductRepository Products { get; }
    IOrderRepository Orders { get; }
    Task<int> CompleteAsync();
}

## UnitOfWork.cs
public class UnitOfWork : IUnitOfWork
{
    private readonly ShopContext _context;

    public IProductRepository Products { get; }
    public IOrderRepository Orders { get; }

    public UnitOfWork(ShopContext context)
    {
        _context = context;
        Products = new ProductRepository(_context);
        Orders = new OrderRepository(_context);
    }

    public async Task<int> CompleteAsync()
    {
        return await _context.SaveChangesAsync();
    }

    public void Dispose()
    {
        _context.Dispose();
    }
}

## Использование + транзакция
Program.cs
using var context = new ShopContext();
context.Database.EnsureCreated();

using var transaction = await context.Database.BeginTransactionAsync();
using var unitOfWork = new UnitOfWork(context);

try
{
    var product = new Product
    {
        Name = "Phone",
        Price = 800
    };

    unitOfWork.Products.Add(product);

    var order = new Order
    {
        CustomerId = 1,
        TotalAmount = 800
    };

    unitOfWork.Orders.Add(order);

    await unitOfWork.CompleteAsync();
    await transaction.CommitAsync();

    Console.WriteLine("Транзакция выполнена успешно.");
}
catch (Exception ex)
{
    await transaction.RollbackAsync();
    Console.WriteLine("Ошибка! Транзакция откатана.");
    Console.WriteLine(ex.Message);
}
