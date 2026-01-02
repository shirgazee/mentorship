### Single Responsibility (принцип единственной ответственности)

У класса должна быть одна причина для изменения. Нормальным языком - класс должен выполнять 1 вещь.

- Пример невыполнения

    Код регистрации выполняет 3 вещи: проверяет емейл, создает пользователя и отправляет ему сообщение.
    
    ```C#
    public class UserService
    {
       public void Register(string email, string password)
       {
          if (!ValidateEmail(email))
             throw new ValidationException("Email is not an email");
             var user = new User(email, password);
    
             SendEmail(new MailMessage("mysite@nowhere.com", email) { Subject="HEllo foo" });
       }
       public virtual bool ValidateEmail(string email)
       {
         return email.Contains("@");
       }
       public bool SendEmail(MailMessage message)
       {
         _smtpClient.Send(message);
       }
    }
    ```
    
    Для выполнения принципа было бы неплохо разделить валидацию, регистрацию и отправку сообщения на разные классы.
    

### Open/Closed Principle (принцип открытости/закрытости)

Класс должен быть открыт для расширения и закрыт для модификации.

Если мы написали класс, то он его логика должна изменяться только если там нашли баги. Если требования расширились, то класс должен изменяться через расширения, например наследование.

- Пример
    
    Логика `AreaCalculator` будет изменяться каждый раз, когда мы будем добавлять новую фигуру, которой нужно будет посчитать площадь:
    
    ```C#
    public class Rectangle{
       public double Height {get;set;}
       public double Wight {get;set; }
    }
    public class Circle{
       public double Radius {get;set;}
    }
    public class AreaCalculator
    {
       public double TotalArea(object[] arrObjects)
       {
          double area = 0;
          Rectangle objRectangle;
          Circle objCircle;
          foreach(var obj in arrObjects)
          {
             if(obj is Rectangle)
             {
                area += obj.Height * obj.Width;
             }
             else
             {
                objCircle = (Circle)obj;
                area += objCircle.Radius * objCircle.Radius * Math.PI;
             }
          }
          return area;
       }
    }
    ```
    
    Можно вынести подсчет площади фигур в сами классы, и использовать наследование для вычисления общей площади:
    
    ```C#
    public abstract class Shape
    {
       public abstract double Area();
    }
    
    public class Rectangle: Shape
    {
       public double Height {get;set;}
       public double Width {get;set;}
       public override double Area()
       {
          return Height * Width;
       }
    }
    public class Circle: Shape
    {
       public double Radius {get;set;}
       public override double Area()
       {
          return Radius * Radus * Math.PI;
       }
    }
    
    public class AreaCalculator
    {
       public double TotalArea(Shape[] arrShapes)
       {
          double area=0;
          foreach(var objShape in arrShapes)
          {
             area += objShape.Area();
          }
          return area;
       }
    }
    ```
    

### Liskov Substitution Principle (принцип подстановки Лисков)

Барбара Ли́сков - это такая тетенька. Принцип говорит, что вместо базового класса должно быть возможно подставить любого его наследника, и все будет работать.

В C# наследник не должен выбрасывать `NotImplementedException` в переопределенных методах базового класса.

- Пример
    
    Реальный пример, вдруг кто спросит, где в .net не выполняется принцип Лисков: [https://github.com/microsoft/referencesource/blob/master/mscorlib/system/collections/objectmodel/readonlycollection.cs](https://github.com/microsoft/referencesource/blob/master/mscorlib/system/collections/objectmodel/readonlycollection.cs)
    
    ```C#
    public interface IMyCollection
    {
      void Add(int item);
      void Remove(int item);
      void Get(int index);
    }
    
    public class MyReadOnlyCollection : IMyCollection 
    {
      private List<int> _collection;
      
      public MyReadOnlyCollection(ICollection<int> collection) 
      {
        _collection = collection;
      }
    
      public void Add(int item)
      {
        throw new NotImplementedException();
      }
    
      public void Remove(int item)
      {
        throw new NotImplementedException();
      }
    
      public void Get(int index)
      {
        return _collection[index];
      }
    }
    ```
    

Или в классе HttpResponse: `Response.Body.Seek(0, SeekOrigin.Begin)` выбросит System.NotSupportedException: Specified method is not supported.

Также нарушением будет добавление более строгих предпроверок или более мягких постпроверок в классе-наследнике:

- Пример
    
    Будет нарушением реализовать такое в наследнике:
    
    ```C#
    public decimal CalculateShippingCost(
        float packageWeightInKilograms,
        Size<float> packageDimensionsInInches,
        RegionInfo destination)
    {
        if (packageWeightInKilograms <= 0f)
          throw new ArgumentOutOfRangeException("packageWeightInKilograms", "Package weight must be positive and non-zero");
    
        return decimal.MinusOne;
    }
    ```
    

### Interface Segregation Principle (принцип разделения интерфейса)

Клиентов нельзя заставлять реализовывать интерфейсы, которые они не должны. По-другому: слишком жирные интерфейсы надо разбивать. Лучше много маленьких интерфейсов, чем 1 здоровый.

- Пример
    
    Пример невыполнения `TeamLead` реализует некоторый набор действий по работе с задачами. Но `Manager` не может реализовать его полностью.
    
    ```C#
    public Interface ILead
    {
       void CreateSubTask();
       void AssginTask();
       void WorkOnTask();
    }
    public class TeamLead : ILead
    {
       public void AssignTask()
       {
       }
       public void CreateSubTask()
       {
       }
       public void WorkOnTask()
       {
       }
    }
    public class Manager: ILead
    {
       public void AssignTask()
       {
       }
       public void CreateSubTask()
       {
       }
       public void WorkOnTask()
       {
          throw new Exception("Manager can't work on Task");
       }
    }
    ```
    
    Можно разбить ILead на 2 интерфейса поменьше:
    
    ```C#
    public interface IProgrammer
    {
       void WorkOnTask();
    }
    public interface ILead
    {
       void AssignTask();
       void CreateSubTask();
    }
    public class TeamLead: IProgrammer, ILead
    {
       public void AssignTask()
       {
       }
       public void CreateSubTask()
       {
       }
       public void WorkOnTask()
       {
       }
    }
    public class Manager: ILead
    {
       public void AssignTask()
       {
       }
       public void CreateSubTask()
       {
       }
    }
    ```
    

### Dependency Inversion Principle (принцип инверсии зависимостей)

Классы должны зависеть от абстракций, а не от реализаций. Детали реализаций может быть невозможно заменить или протестировать.

- Пример
    
    ```C#
    public class User
    {
        private EmailSender emailSender;
    
        public User()
        {
            emailSender = new EmailSender();
        }
    
        public void SendEmail(string email, string message)
        {
            emailSender.Send(email, message);
        }
    }
    ```
    
    Такой класс не протестировать Unit-тестом, нужен интеграционный. Чтобы соблюсти принцип, нужно заменить `EmailSender` на интерфейс `IEmailSender`, который уже можно замокать.
    
