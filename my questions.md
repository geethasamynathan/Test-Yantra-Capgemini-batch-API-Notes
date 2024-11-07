1) Given the following code, what will be the output?

```csharp

class Program
{
    static void Main(string[] args)
    {
        int x = 5;
        int y = 10;
        int result = Add(x, y);
        Console.WriteLine(result);
    }

    static int Add(int a, int b)
    {
        return a + b;
    }
}
```
A. 5\ B. 10\ C. 15\ D. Error

2) In a scenario where you need to handle a collection of unique items that can be accessed by key, which collection would you use?

A. List\ B. Dictionary\ C. Queue\ D. Array

A Dictionary allows you to store and access items by key, and each key in a Dictionary is unique.

3) Consider this scenario: You need to create a class that allows only one instance of itself to be created. Which design pattern would you use?

A. Singleton\ B. Factory\ C. Observer\ D. Decorator