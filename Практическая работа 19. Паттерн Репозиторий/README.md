## Практическая работа №19
## Тема: Паттерн «Репозиторий»
## Вариант: Product (Id, Name, Price)

## Модель сущности

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

## DbContext

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
                "Server=.;Database=ShopDb;Trusted_Connection=True;TrustServerCertificate=True;"
            );
        }
    }
}

## Интерфейс репозитория

## Repositories/IRepository.cs

namespace Project.Repositories
{
    public interface IRepository<T> where T : class
    {
        IEnumerable<T> GetAll();
        T GetById(int id);
        void Add(T entity);
        void Update(T entity);
        void Delete(T entity);
    }
}

## Реализация репозитория

## Repositories/Repository.cs

using Microsoft.EntityFrameworkCore;
using Project.Data;

namespace Project.Repositories
{
    public class Repository<T> : IRepository<T> where T : class
    {
        private readonly AppDbContext _context;
        private readonly DbSet<T> _dbSet;

        public Repository(AppDbContext context)
        {
            _context = context;
            _dbSet = context.Set<T>();
        }

        public IEnumerable<T> GetAll()
        {
            return _dbSet.ToList();
        }

        public T GetById(int id)
        {
            return _dbSet.Find(id);
        }

        public void Add(T entity)
        {
            _dbSet.Add(entity);
            _context.SaveChanges();
        }

        public void Update(T entity)
        {
            _dbSet.Update(entity);
            _context.SaveChanges();
        }

        public void Delete(T entity)
        {
            _dbSet.Remove(entity);
            _context.SaveChanges();
        }
    }
}

## Использование репозитория (CRUD)

## Program.cs

using Project.Data;
using Project.Models;
using Project.Repositories;

using var context = new AppDbContext();
IRepository<Product> productRepository = new Repository<Product>(context);

// CREATE
var product = new Product
{
    Name = "Laptop",
    Price = 90000
};
productRepository.Add(product);
Console.WriteLine("Продукт добавлен");

// READ
var products = productRepository.GetAll();
Console.WriteLine("Список продуктов:");
foreach (var p in products)
{
    Console.WriteLine($"{p.Id}. {p.Name} - {p.Price} руб.");
}

// UPDATE
product.Price = 85000;
productRepository.Update(product);
Console.WriteLine("Цена обновлена");

// DELETE
productRepository.Delete(product);
Console.WriteLine("Продукт удалён");
