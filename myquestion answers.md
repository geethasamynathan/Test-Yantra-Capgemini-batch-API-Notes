Scenario: Base and Derived Classes

You have a base class Employee with properties Name and EmployeeID, and a method GetDetails(). You create a derived class Manager that adds a property Department and overrides the GetDetails() method to include the department information.

Which of the following correctly represents the Manager class?

A.

csharp

Copy
class Manager : Employee
{
    public string Department { get; set; }

    public override string GetDetails()
    {
        return base.GetDetails() + ", Department: " + Department;
    }
}
B.

csharp

Copy
class Manager : Employee
{
    public string Department { get; set; }

    public string GetDetails()
    {
        return base.GetDetails() + ", Department: " + Department;
    }
}
C.

csharp

Copy
class Manager : Employee
{
    public string Department { get; set; }

    public new string GetDetails()
    {
        return base.GetDetails() + ", Department: " + Department;
    }
}
D.

```csharp

class Manager : Employee
{
    public string Department { get; set; }

    public string GetDetails()
    {
        return "Department: " + Department;
    }
}
```
Answer: A. The GetDetails() method is correctly overridden in the Manager class to include the department information.

Scenario: Using Interfaces

Consider an interface IAppliance with a method TurnOn(). You have a base class Machine with a method Start(). You want to create a class WashingMachine that inherits from Machine and implements IAppliance.

Which of the following correctly represents the WashingMachine class?

A.

```csharp

class WashingMachine : Machine, IAppliance
{
    public void TurnOn()
    {
        Console.WriteLine("Washing Machine is now ON.");
    }
}
```
B.

```csharp

class WashingMachine : Machine, IAppliance
{
    public void Start()
    {
        Console.WriteLine("Washing Machine is now ON.");
    }
}```
C.

```csharp

class WashingMachine : IAppliance
{
    public void TurnOn()
    {
        Console.WriteLine("Washing Machine is now ON.");
    }
}```
D.

```csharp

class WashingMachine : Machine, IAppliance
{
    public void Start()
    {
        Console.WriteLine("Starting the machine...");
    }

    public void TurnOn()
    {
        Console.WriteLine("Washing Machine is now ON.");
    }
}
```
Answer: D. The WashingMachine class correctly inherits from Machine and implements the IAppliance interface, providing both the Start() and TurnOn() methods.

Scenario: Abstract Classes

You have an abstract class Shape with an abstract method Draw(). You want to create a class Circle that inherits from Shape and implements the Draw() method.

Which of the following correctly represents the Circle class?

A.

```csharp

class Circle : Shape
{
    public override void Draw()
    {
        Console.WriteLine("Drawing a Circle.");
    }
}
```
B.

```csharp

abstract class Circle : Shape
{
    public override void Draw()
    {
        Console.WriteLine("Drawing a Circle.");
    }
}
```
C.

```csharp

class Circle : Shape
{
    public void Draw()
    {
        Console.WriteLine("Drawing a Circle.");
    }
}
```
D.

```csharp

class Circle : Shape
{
    public new void Draw()
    {
        Console.WriteLine("Drawing a Circle.");
    }
}
```
Answer: A. The Circle class correctly inherits from Shape and overrides the abstract Draw() method.