Testing C# syntax

```csharp
public class ProductViewModel {
  [Price(MinPrice = 1.99)]
  public double Price { get; set; }

  [Required]
  public string Title { get; set; }
}
```