# C# & OOP Interview Guide with Use Cases

## 1️⃣ Value Types vs Reference Types

### Interview Questions

**Q1: What's the difference between value types and reference types in C#?**
- **Answer**: Value types store data directly on the stack and are copied when assigned. Reference types store a reference on the stack pointing to data on the heap, so multiple variables can reference the same object.

**Q2: What happens when you pass a value type vs reference type to a method?**
- **Answer**: Value types are passed by value (copied), so changes inside the method don't affect the original. Reference types pass the reference, so changes to the object's properties affect the original object.

**Q3: What is boxing and unboxing? Why is it a performance concern?**
- **Answer**: Boxing converts a value type to object (heap allocation). Unboxing extracts the value type back. It's costly because it involves heap allocation and type checking.

### Real-World Use Cases

**Use Case 1: E-commerce Order Line Items**
```csharp
// Good: Struct for immutable line item data
public struct OrderLineItem
{
    public int ProductId { get; }
    public decimal Price { get; }
    public int Quantity { get; }
    
    public OrderLineItem(int productId, decimal price, int quantity)
    {
        ProductId = productId;
        Price = price;
        Quantity = quantity;
    }
}

// Usage: Each line item is independent
var item1 = new OrderLineItem(101, 29.99m, 2);
var item2 = item1; // Copy created
// Modifying item2 won't affect item1
```

**Use Case 2: Shopping Cart (Reference Type)**
```csharp
// Good: Class for mutable shopping cart
public class ShoppingCart
{
    public List<OrderLineItem> Items { get; } = new();
    public string UserId { get; set; }
    
    public void AddItem(OrderLineItem item) => Items.Add(item);
}

// Usage: Multiple references to same cart
var cart = new ShoppingCart { UserId = "user123" };
var cartReference = cart; // Both point to same object
cartReference.AddItem(new OrderLineItem(101, 29.99m, 1));
// cart.Items.Count is now 1 (same object)
```

---

## 2️⃣ Structs vs Classes

### Interview Questions

**Q1: When should you use a struct instead of a class?**
- **Answer**: Use structs for small, immutable data types that represent a single value (like Point, Color, DateTime). Use classes for objects with behavior, mutable state, or when inheritance is needed.

**Q2: Can a struct inherit from another struct or class?**
- **Answer**: No, structs cannot inherit from other structs or classes. They can only implement interfaces. All structs implicitly inherit from System.ValueType.

**Q3: What are the limitations of structs?**
- **Answer**: No parameterless constructor (before C# 10), can't inherit, copying large structs is expensive, and they must be fully initialized.

### Real-World Use Cases

**Use Case 1: 2D Game Coordinates**
```csharp
// Perfect for struct: Small, immutable position data
public struct Position
{
    public int X { get; }
    public int Y { get; }
    
    public Position(int x, int y)
    {
        X = x;
        Y = y;
    }
    
    public Position Move(int deltaX, int deltaY) 
        => new Position(X + deltaX, Y + deltaY);
}

// Usage in game
var playerPos = new Position(10, 20);
var newPos = playerPos.Move(5, 0); // Creates new struct
```

**Use Case 2: Game Character (Class)**
```csharp
// Good: Class for complex entity with behavior
public class GameCharacter
{
    public string Name { get; set; }
    public Position CurrentPosition { get; set; }
    public int Health { get; set; }
    public List<string> Inventory { get; } = new();
    
    public void TakeDamage(int damage)
    {
        Health = Math.Max(0, Health - damage);
    }
    
    public void MoveTo(Position newPosition)
    {
        CurrentPosition = newPosition;
    }
}
```

---

## 3️⃣ Records & Immutability

### Interview Questions

**Q1: What is a record in C# and when should you use it?**
- **Answer**: A record is a reference type designed for immutable data with value-based equality. Use it for DTOs, data transfer objects, and when you want immutability with easy cloning via `with` expressions.

**Q2: What's the difference between a record and a class?**
- **Answer**: Records provide value-based equality by default, have built-in ToString() implementation, support `with` expressions for non-destructive mutation, and are ideal for immutable data.

**Q3: What's the downside of using records everywhere?**
- **Answer**: Frequent updates create new instances (memory overhead), and they're not suitable for entities that need to maintain identity or have frequent mutable operations.

### Real-World Use Cases

**Use Case 1: API Response DTOs**
```csharp
// Perfect for records: Immutable data transfer
public record UserDto(
    int Id, 
    string Username, 
    string Email, 
    DateTime CreatedAt
);

public record PaginatedResponse<T>(
    IReadOnlyList<T> Data,
    int Page,
    int PageSize,
    int TotalCount
);

// Usage
var user = new UserDto(1, "john_doe", "john@example.com", DateTime.Now);
var updatedUser = user with { Email = "newemail@example.com" };
```

**Use Case 2: Configuration Settings**
```csharp
// Good: Immutable configuration
public record DatabaseConfig(
    string ConnectionString,
    int MaxPoolSize,
    int CommandTimeout
)
{
    public DatabaseConfig WithTimeout(int timeout) 
        => this with { CommandTimeout = timeout };
}

// Usage: Thread-safe configuration
var config = new DatabaseConfig(
    "Server=localhost;Database=MyDb", 
    100, 
    30
);
var devConfig = config with { ConnectionString = "Server=dev-server;Database=MyDb" };
```

**Use Case 3: Event Sourcing Events**
```csharp
// Excellent for records: Immutable domain events
public record OrderPlacedEvent(
    Guid OrderId,
    string UserId,
    decimal TotalAmount,
    DateTime PlacedAt
);

public record OrderShippedEvent(
    Guid OrderId,
    string TrackingNumber,
    DateTime ShippedAt
);
```

---

## 4️⃣ Static vs Non-static Classes

### Interview Questions

**Q1: What's the purpose of a static class?**
- **Answer**: Static classes cannot be instantiated and contain only static members. They're used for utility functions, extension methods, and constants that don't require state.

**Q2: Why are static classes hard to test?**
- **Answer**: They can't be mocked or implement interfaces, creating tight coupling. They also maintain state across the entire application lifetime, which can cause issues in unit tests.

**Q3: When should you avoid using static classes?**
- **Answer**: When you need dependency injection, polymorphism, testability, or when the class needs to maintain instance-specific state.

### Real-World Use Cases

**Use Case 1: Utility Helpers (Good for Static)**
```csharp
// Perfect for static: Stateless utility functions
public static class StringHelpers
{
    public static string ToTitleCase(string input)
    {
        if (string.IsNullOrEmpty(input)) return input;
        return char.ToUpper(input[0]) + input.Substring(1).ToLower();
    }
    
    public static bool IsValidEmail(string email)
    {
        return !string.IsNullOrEmpty(email) && email.Contains("@");
    }
}

// Usage
var title = StringHelpers.ToTitleCase("hello world");
```

**Use Case 2: Email Service (Good for Non-static)**
```csharp
// Good: Non-static for testability and DI
public interface IEmailService
{
    Task SendEmailAsync(string to, string subject, string body);
}

public class EmailService : IEmailService
{
    private readonly IConfiguration _config;
    private readonly ILogger<EmailService> _logger;
    
    public EmailService(IConfiguration config, ILogger<EmailService> logger)
    {
        _config = config;
        _logger = logger;
    }
    
    public async Task SendEmailAsync(string to, string subject, string body)
    {
        var smtpServer = _config["SmtpServer"];
        _logger.LogInformation("Sending email to {To}", to);
        // Send email logic
    }
}

// Testable, mockable, supports DI
```

**Use Case 3: Constants and Enums**
```csharp
// Perfect for static: Application-wide constants
public static class AppConstants
{
    public const int MaxLoginAttempts = 3;
    public const string DefaultCurrency = "USD";
    public static readonly TimeSpan SessionTimeout = TimeSpan.FromMinutes(30);
}
```

---

## 5️⃣ Delegates

### Interview Questions

**Q1: What is a delegate in C#?**
- **Answer**: A delegate is a type-safe function pointer that can reference methods with a specific signature. It allows passing methods as parameters and enables callbacks.

**Q2: What's the difference between Action, Func, and Predicate?**
- **Answer**: `Action` has no return value, `Func` returns a value, and `Predicate` returns a bool. They're built-in generic delegates that eliminate the need to declare custom delegates.

**Q3: When would you create a custom delegate instead of using Action/Func?**
- **Answer**: When you want a more meaningful name (e.g., `DataValidationHandler`) or when the signature is complex and deserves semantic clarity.

### Real-World Use Cases

**Use Case 1: Payment Processing with Multiple Gateways**
```csharp
// Delegate for payment processing strategy
public delegate PaymentResult ProcessPaymentDelegate(
    decimal amount, 
    string paymentMethod
);

public class PaymentService
{
    private readonly Dictionary<string, ProcessPaymentDelegate> _processors = new();
    
    public void RegisterProcessor(string name, ProcessPaymentDelegate processor)
    {
        _processors[name] = processor;
    }
    
    public PaymentResult ProcessPayment(string gateway, decimal amount, string method)
    {
        if (_processors.TryGetValue(gateway, out var processor))
            return processor(amount, method);
        
        throw new InvalidOperationException($"Gateway {gateway} not found");
    }
}

// Usage
var service = new PaymentService();
service.RegisterProcessor("stripe", ProcessStripePayment);
service.RegisterProcessor("paypal", ProcessPayPalPayment);

var result = service.ProcessPayment("stripe", 99.99m, "credit_card");
```

**Use Case 2: Data Transformation Pipeline**
```csharp
// Using Func for data transformation
public class DataPipeline<T>
{
    private readonly List<Func<T, T>> _transformations = new();
    
    public DataPipeline<T> AddTransformation(Func<T, T> transform)
    {
        _transformations.Add(transform);
        return this;
    }
    
    public T Execute(T input)
    {
        return _transformations.Aggregate(input, (current, transform) 
            => transform(current));
    }
}

// Usage
var pipeline = new DataPipeline<string>()
    .AddTransformation(s => s.Trim())
    .AddTransformation(s => s.ToUpper())
    .AddTransformation(s => s.Replace(" ", "_"));

var result = pipeline.Execute("  hello world  "); // "HELLO_WORLD"
```

---

## 6️⃣ Events

### Interview Questions

**Q1: What's the difference between a delegate and an event?**
- **Answer**: Events are built on delegates but provide encapsulation. Only the declaring class can invoke an event, while subscribers can only add/remove handlers. This prevents external code from clearing all subscribers or invoking the event.

**Q2: What's a common memory leak issue with events?**
- **Answer**: If subscribers don't unsubscribe from events, the publisher holds references to them, preventing garbage collection. This is especially problematic with long-lived publishers and short-lived subscribers.

**Q3: When should you use events vs other patterns like observers or message buses?**
- **Answer**: Use events for in-process, tightly-coupled scenarios within a single application. Use message buses for distributed systems or loosely-coupled microservices.

### Real-World Use Cases

**Use Case 1: Order Processing System**
```csharp
// Event-driven order processing
public class Order
{
    public event EventHandler<OrderStatusChangedEventArgs> StatusChanged;
    
    private OrderStatus _status;
    public OrderStatus Status
    {
        get => _status;
        private set
        {
            if (_status != value)
            {
                var oldStatus = _status;
                _status = value;
                OnStatusChanged(new OrderStatusChangedEventArgs(oldStatus, value));
            }
        }
    }
    
    protected virtual void OnStatusChanged(OrderStatusChangedEventArgs e)
    {
        StatusChanged?.Invoke(this, e);
    }
    
    public void Ship() => Status = OrderStatus.Shipped;
}

public class OrderStatusChangedEventArgs : EventArgs
{
    public OrderStatus OldStatus { get; }
    public OrderStatus NewStatus { get; }
    
    public OrderStatusChangedEventArgs(OrderStatus oldStatus, OrderStatus newStatus)
    {
        OldStatus = oldStatus;
        NewStatus = newStatus;
    }
}

// Subscribers
public class EmailNotificationService
{
    public void Subscribe(Order order)
    {
        order.StatusChanged += OnOrderStatusChanged;
    }
    
    private void OnOrderStatusChanged(object sender, OrderStatusChangedEventArgs e)
    {
        if (e.NewStatus == OrderStatus.Shipped)
            Console.WriteLine("Sending shipped notification email...");
    }
}

public class InventoryService
{
    public void Subscribe(Order order)
    {
        order.StatusChanged += OnOrderStatusChanged;
    }
    
    private void OnOrderStatusChanged(object sender, OrderStatusChangedEventArgs e)
    {
        if (e.NewStatus == OrderStatus.Shipped)
            Console.WriteLine("Updating inventory...");
    }
}
```

**Use Case 2: Real-time Dashboard Updates**
```csharp
// Stock price monitoring with events
public class StockMonitor
{
    public event EventHandler<PriceChangedEventArgs> PriceChanged;
    
    private decimal _currentPrice;
    
    public void UpdatePrice(decimal newPrice)
    {
        var oldPrice = _currentPrice;
        _currentPrice = newPrice;
        
        PriceChanged?.Invoke(this, new PriceChangedEventArgs(oldPrice, newPrice));
    }
}

public class PriceChangedEventArgs : EventArgs
{
    public decimal OldPrice { get; }
    public decimal NewPrice { get; }
    public decimal ChangePercent => ((NewPrice - OldPrice) / OldPrice) * 100;
    
    public PriceChangedEventArgs(decimal oldPrice, decimal newPrice)
    {
        OldPrice = oldPrice;
        NewPrice = newPrice;
    }
}

// Usage
var monitor = new StockMonitor();
monitor.PriceChanged += (sender, e) => 
    Console.WriteLine($"Price changed by {e.ChangePercent:F2}%");
monitor.PriceChanged += (sender, e) =>
{
    if (Math.Abs(e.ChangePercent) > 5)
        Console.WriteLine("Alert: Significant price movement!");
};
```

---

## 7️⃣ Lambdas

### Interview Questions

**Q1: What is a lambda expression?**
- **Answer**: A lambda is an anonymous function that can be used to create delegates or expression trees. Syntax: `(parameters) => expression` or `(parameters) => { statements }`.

**Q2: What's the difference between a lambda and an anonymous method?**
- **Answer**: Lambdas have shorter syntax and can be converted to expression trees. Anonymous methods use `delegate` keyword and cannot be converted to expression trees.

**Q3: What's a closure in the context of lambdas?**
- **Answer**: A closure occurs when a lambda captures variables from its enclosing scope. The captured variables remain accessible even after the enclosing method returns.

### Real-World Use Cases

**Use Case 1: LINQ Queries**
```csharp
// Lambda with LINQ for data filtering
public class ProductService
{
    private readonly List<Product> _products;
    
    public IEnumerable<Product> GetDiscountedProducts(decimal minDiscount)
    {
        return _products
            .Where(p => p.Discount >= minDiscount)
            .OrderByDescending(p => p.Discount)
            .Select(p => new 
            { 
                p.Name, 
                p.Price, 
                FinalPrice = p.Price * (1 - p.Discount) 
            });
    }
    
    public decimal GetAveragePrice(string category)
    {
        return _products
            .Where(p => p.Category == category)
            .Average(p => p.Price);
    }
}
```

**Use Case 2: Event Handlers**
```csharp
// Lambda for inline event subscription
public class UserInterface
{
    public void SetupButtons()
    {
        var saveButton = new Button();
        var cancelButton = new Button();
        
        // Inline lambda event handlers
        saveButton.Click += (sender, e) => 
        {
            Console.WriteLine("Saving data...");
            SaveData();
        };
        
        cancelButton.Click += (sender, e) => 
        {
            Console.WriteLine("Operation cancelled");
            ResetForm();
        };
    }
    
    private void SaveData() { }
    private void ResetForm() { }
}
```

**Use Case 3: Retry Logic with Lambda**
```csharp
// Lambda for flexible retry mechanism
public class RetryHelper
{
    public static T ExecuteWithRetry<T>(
        Func<T> operation, 
        int maxRetries = 3, 
        TimeSpan? delay = null)
    {
        var retryDelay = delay ?? TimeSpan.FromSeconds(1);
        
        for (int i = 0; i < maxRetries; i++)
        {
            try
            {
                return operation();
            }
            catch (Exception ex) when (i < maxRetries - 1)
            {
                Console.WriteLine($"Attempt {i + 1} failed: {ex.Message}");
                Thread.Sleep(retryDelay);
            }
        }
        
        return operation(); // Final attempt without catching
    }
}

// Usage
var result = RetryHelper.ExecuteWithRetry(() => 
{
    // Some operation that might fail
    return CallExternalApi();
}, maxRetries: 5);
```

---

## 8️⃣ LINQ

### Interview Questions

**Q1: What is deferred execution in LINQ?**
- **Answer**: LINQ queries don't execute when defined, but when enumerated (e.g., foreach, ToList(), Count()). This allows building complex queries efficiently but can cause unexpected behavior if the source changes.

**Q2: What's the difference between IEnumerable and IQueryable in LINQ?**
- **Answer**: `IEnumerable` executes in-memory (LINQ to Objects), while `IQueryable` builds expression trees for external data sources like databases (LINQ to SQL/EF), allowing query translation.

**Q3: When should you avoid LINQ for performance reasons?**
- **Answer**: In tight loops with large datasets, hot paths in performance-critical code, or when you need fine-grained control over iterations. Custom loops can be faster.

### Real-World Use Cases

**Use Case 1: E-commerce Report Generation**
```csharp
public class SalesReportService
{
    private readonly List<Order> _orders;
    
    public SalesReport GenerateMonthlyReport(int year, int month)
    {
        var monthlyOrders = _orders
            .Where(o => o.OrderDate.Year == year && o.OrderDate.Month == month)
            .ToList();
        
        return new SalesReport
        {
            TotalRevenue = monthlyOrders.Sum(o => o.TotalAmount),
            OrderCount = monthlyOrders.Count,
            AverageOrderValue = monthlyOrders.Average(o => o.TotalAmount),
            TopProducts = monthlyOrders
                .SelectMany(o => o.Items)
                .GroupBy(i => i.ProductId)
                .Select(g => new 
                { 
                    ProductId = g.Key, 
                    TotalSold = g.Sum(i => i.Quantity),
                    Revenue = g.Sum(i => i.Price * i.Quantity)
                })
                .OrderByDescending(p => p.Revenue)
                .Take(10)
                .ToList(),
            CustomersBySegment = monthlyOrders
                .GroupBy(o => o.TotalAmount switch
                {
                    < 50 => "Low",
                    < 200 => "Medium",
                    _ => "High"
                })
                .ToDictionary(g => g.Key, g => g.Count())
        };
    }
}
```

**Use Case 2: User Permission Filtering**
```csharp
public class DocumentService
{
    private readonly List<Document> _documents;
    
    public IEnumerable<Document> GetAccessibleDocuments(
        User user, 
        string searchTerm = null,
        DocumentType? type = null)
    {
        var query = _documents.AsQueryable();
        
        // Filter by permissions
        query = query.Where(d => 
            d.OwnerId == user.Id || 
            d.SharedWith.Contains(user.Id) ||
            user.Roles.Contains("Admin"));
        
        // Optional search
        if (!string.IsNullOrEmpty(searchTerm))
        {
            query = query.Where(d => 
                d.Title.Contains(searchTerm) || 
                d.Content.Contains(searchTerm));
        }
        
        // Optional type filter
        if (type.HasValue)
        {
            query = query.Where(d => d.Type == type.Value);
        }
        
        return query
            .OrderByDescending(d => d.LastModified)
            .Take(100)
            .ToList();
    }
}
```

**Use Case 3: Data Transformation Pipeline**
```csharp
public class DataTransformService
{
    public List<CustomerAnalytics> TransformCustomerData(
        List<RawCustomerData> rawData)
    {
        return rawData
            .Where(c => c.IsActive && !string.IsNullOrEmpty(c.Email))
            .GroupBy(c => c.CustomerId)
            .Select(g => new CustomerAnalytics
            {
                CustomerId = g.Key,
                FirstName = g.First().FirstName,
                LastName = g.First().LastName,
                TotalPurchases = g.Sum(c => c.PurchaseAmount),
                PurchaseCount = g.Count(),
                AveragePurchase = g.Average(c => c.PurchaseAmount),
                FirstPurchaseDate = g.Min(c => c.PurchaseDate),
                LastPurchaseDate = g.Max(c => c.PurchaseDate),
                FavoriteCategory = g
                    .GroupBy(c => c.Category)
                    .OrderByDescending(cg => cg.Count())
                    .First()
                    .Key
            })
            .OrderByDescending(c => c.TotalPurchases)
            .ToList();
    }
}
```

---

## 🎯 Key Interview Tips

### Common Pitfalls to Discuss

1. **Boxing/Unboxing**: Avoid putting value types in `object` or `ArrayList`
2. **Event Memory Leaks**: Always unsubscribe from events in Dispose()
3. **LINQ Performance**: Use `Any()` instead of `Count() > 0`, materialize queries when needed
4. **Closure Traps**: Be careful capturing loop variables in lambdas
5. **Immutability Overhead**: Don't use records for frequently mutated objects

### Best Practices to Mention

1. **Prefer composition over inheritance**
2. **Use interfaces for abstractions, not static classes**
3. **Make immutability the default when possible**
4. **Use LINQ for readability, loops for performance-critical paths**
5. **Design for testability from the start**

### Advanced Topics to Explore

- Expression trees and dynamic query building
- Async/await with delegates and events
- Span<T> and Memory<T> for high-performance scenarios
- Record structs (C# 10+)
- Pattern matching with records
