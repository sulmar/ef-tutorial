# Przewodnik Entity Framework 6.2

## Instalacja biblioteki

~~~ bash
PM> Install-Package EntityFramework
~~~

## Utworzenie kontekstu

~~~ csharp
public class MyContext : DbContext
{
    public MyContext()
      :base("MyDbConnection")
    { }

    public DbSet<Customer> Customers { get; set; }
    public DbSet<Order> Orders { get; set; }
}
~~~

## Połączenie do bazy danych

### Algorytm wyszukiwania bazy danych

1. Jeśli w konstruktorze DbContext podano nazwę połączenia to szuka jej w pliku konfiguracynym app.config w sekcji _connectionStrings_ 

2. Jeśli użyto domyślnego konstruktora DbContext to szuka pliku konfiguracynym app.config nazwy klasy DbContext w sekcji _connectionStrings_  

3. Szuka instancji SQL Express 

4. Szuka bazy danych LocalDb o adresie (localdb)\mssqllocaldb



### Wyświetlenie parametrów połączenia

~~~ csharp
Console.WriteLine(context.Database.Connection.ConnectionString);
~~~



- Azure Data Studio

https://docs.microsoft.com/en-us/sql/azure-data-studio/download?view=sql-server-2017



## Przydatne polecenia PMC
- ``` Enable-Migrations ``` - włączenie migracji
- ``` Add-Migration {migration} ``` - utworzenie migracji
- ``` Update-Database ``` - aktualizacja bazy danych do najnowszej wersji
- ``` Update-Database -script ``` - wygenerowanie skryptu do aktualizacji bazy danych do najnowszej wersji
- ``` Update-Database -verbose ``` - aktualizacja bazy danych do najnowszej wersji + wyświetlanie logu
- ``` Update-Database -TargetMigration: {migration} ``` - aktualizacja bazy danych do wskazanej migracji
- ``` Update-Database -SourceMigration: {migrationA} -TargetMigration: {migrationB}  ```  - aktualizacja bazy danych pomiędzy migracjami


## DbContext
Klasa DbContext jest główną częścią Entity Framework. Instacja DbContext reprezentuje sesję z bazą danych.


Models.cs

~~~ csharp

public class Order
{
    public int OrderId { get; set; }   
    public string OrderNumber { get; set; }  
    public DateTime OrderDate { get; set; }
    public DateTime DeliveryDate? { get; set; }
    public Customer Customer { get; set; }
}

public class Customer
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public bool IsDeleted { get; set; }
}

~~~


MyContext.cs

~~~ csharp
public class MyContext : DbContext
{
    public DbSet<Customer> Customers { get; set; }
    public DbSet<Order> Orders { get; set; }

    public CustomersContext() 
        : base("MyDbConnection")
    {
    }
}
~~~

~~~ csharp

using (var context = new MyContext(optionsBuilder.Options))
{
  // do stuff
}
~~~

DbContext umożliwia następujące zadania:
 1. Zarządzanie połączeniem z bazą danych
 2. Konfiguracja modelu i relacji
 3. Odpytywanie bazy danych
 4. Zapisywanie danych do bazy danych
 5. Śledzenie zmian
 6. Cache'owanie
 7. Zarządzanie transakcjami

## Właściwości DbContext
| Metoda | Użycie |
|---|---|
| ChangeTracker | Dostarcza informacje i operacje do śledzenie obiektów  |
| Database | Dostarcza informacje o bazie danych i umożliwia operacje na bazie danych |

## Fabryka
Jeśli korzystamy z migracji, a konsuktor naszej klasy DbContext posiada parametr(y) nalezy utworzyć fabrykę:

~~~ csharp
    
    public class MyContextFactory : IDbContextFactory<TransportContext>
    {
        public MyContext Create()
        {
            return new MyContext(new TransportDbInitializer());
        }
    }
    
~~~


## Konwencja relacji Jeden-do-wielu

### Konwencja 1
Encja zawiera navigation property.

``` csharp
public class Order
{
    public int OrderId { get; set; }   
    public string OrderNumber { get; set; }  

    public Customer Customer { get; set; } // Navigation property
}

public class Customer
{
    public int CustomerId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
}

```
Zamówienie zawiera referencje do navigation property typu klient. EF utworzy shadow property CustomerId w modelu koncepcyjnym, które będzie mapowane do kolumny CustomerId w tabeli Orders.

### Konwencja 2
Encja zawiera kolekcję.

``` csharp
public class Order
{
    public int OrderId { get; set; }       
    public string OrderNumber { get; set; } 
}

public class Customer
{
    public int CustomerId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }

    public List<Order> Orders { get; set; }
}

```

W bazie danych będzie taki sam rezultat jak w przypadku konwencji 1.

### Konwencja 3

Relacja zawiera navigation property po obu stronach. W rezultacie otrzymujemy połączenie konwencji 1 i 2.


``` csharp
public class Order
{
    public int OrderId { get; set; }       
    public string OrderNumber { get; set; } 

    public Customer Customer { get; set; } // Navigation property
}

public class Customer
{
    public int CustomerId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }

    public List<Order> Orders { get; set; }
}

```

### Konwencja 4
Konwencja z uzyciem wlasciwosci foreign key


``` csharp
public class Order
{
    public int OrderId { get; set; }       
    public string OrderNumber { get; set; } 

    public int CustomerId { get; set; }  // Foreign key property
    public Customer Customer { get; set; } // Navigation property
}

public class Customer
{
    public int CustomerId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }

    public List<Order> Orders { get; set; }
}

```


## Konwencja relacji Jeden-do-jeden

``` csharp
public class Order
{
    public int OrderId { get; set; }       
    public string OrderNumber { get; set; } 

    public Payment Payment { get; set; } // Navigation property
}

public class Payment
{
    public int PaymentId { get; set; }
    public decimal Amount { get; set; }

    public int OrderId { get; set; }
    public Order Order { get; set; }
}
```

## Konwencja relacji wiele-do-wielu


``` csharp
public class User
{
    public int Id { get; set; }       
    public string Login { get; set; } 

    public ICollection<Role> Roles { get; set; } // Navigation property
}

public class Role
{
    public int Id { get; set; }
    public string Name { get; set; }

    public ICollection<User> Users { get; set; }  // Navigation property
}
```

Zostanie automatycznie utworzona tabela pośrednia

## Konfiguracja relacji Jeden-do-wielu z uzyciem Fluent API


``` csharp
   protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<Order>()
                .HasOne<Customer>()
                .WithMany(c=>c.Orders)
                .HasForeignKey(p=>p.CustomerId);
``` 


Alternatywnie mozna wyjsc od drugiej strony
``` csharp
            modelBuilder.Entity<Customer>()
                .HasMany(c=>c.Orders)
                .WithOne(o=>o.Customer)
                .HasForeignKey(o=>o.CustomerId);


        }
```

## Konfiguracja kaskadowego usuwania z Fluent API

``` csharp
modelBuilder.Entity<Customer>()
                .HasMany(c=>c.Orders)
                .WithOne(o=>o.Customer)
                .HasForeignKey(o=>o.CustomerId)
                .OnDelete(DeleteBehavior.Cascade);
```

Rodzaje:
- Cascade - usuwa wszystkie encje wraz z encją nadrzędną
- ClientSetNull - klucze obce w encjach zaleznych będą ustawione na null
- Restrict - blokuje kaskadowe usuwanie
- SetNull - klucze obce w encjach zaleznych będą ustawione na null


## Konfiguracja jeden-do-jeden z Fluent API


``` csharp
 modelBuilder.Entity<Order>()
                .HasOne<Payment>()
                .WithOne(p=>p.Order)
                .HasForeignKey<Payment>(p=>p.PaymentId);
```

## Konfiguracja wiele-do-wiele z Fluent API

### Podstawowa konfiguracja

``` csharp

modelBuilder.Entity<User>()
                .HasMany(e => e.Roles)
                .WithMany(e => e.Users);
```

Powstanie tabela pośrednicząca o nazwie UserRoles


Jeśli zamienimy miejscami pola otrzymamy inną nazwę tabeli:

``` csharp

modelBuilder.Entity<Role>()
                .HasMany(e => e.Users)
                .WithMany(e => e.Roles);
```
Powstanie tabela pośrednicząca o nazwie RoleUsers

### Własna nazwy tabeli

``` csharp
modelBuilder.Entity<User>()
                .HasMany(e => e.Roles)
                .WithMany(e => e.Users)
                .Map(m=>
                {
                    m.ToTable("UsersInRoles");
                });
```

Powstanie tabela pośrednicząca o nazwie UsersInRoles

### Własne klucze obce


``` csharp
modelBuilder.Entity<User>()
                .HasMany(e => e.Roles)
                .WithMany(e => e.Users)
                .Map(m=>
                {
                    m.MapLeftKey("UserId");
                    m.MapRightKey("RoleId");
                });

```


## Śledzenie (Tracking)

### AutoDetectChanges

Domyślnie śledzenie zmian jest włączone. W celu zwiększenia wydajności, zwłaszcza przy dodawaniu dużej ilości encji warto wyłączyć automatyczne wykrywanie zmian:


~~~ csharp
using (var context = new MyContext())
{
    context.Configuration.AutoDetectChangesEnabled = false;
}
~~~

Pamiętaj o wywołaniu metody _DetectChanges()_ przed _SaveChanges()_

Przykład:
~~~ csharp
using (var context = new MyContext())
{

            using (var context = new MyContext())
            {
                try
                {
                    context.Configuration.AutoDetectChangesEnabled = false;
 
                    foreach (var product in context.Products)
                    {
                        product.UnitPrice = 1.0m;
                    }
                }
                finally
                {
                    context.Configuration.AutoDetectChangesEnabled = true;
                }
 
                context.ChangeTracker.DetectChanges();
                context.SaveChanged();
            }
        }
~~~

### Lokalne wyłączenie śledzenia

~~~ csharp
using (var context = new MyContext())
{
    var blogs = context.Customers
        .AsNoTracking()
        .ToList();
}
~~~



### Pobranie informacji o encjach

~~~ csharp
 Console.WriteLine(
   $"Tracked Entities: {context.ChangeTracker.Entries().Count()}");

foreach (var entry in context.ChangeTracker.Entries())
{
    Console.WriteLine($"Entity: {entry.Entity.GetType().Name}, 
                        State: {entry.State.ToString()} ");
}

~~~



## Praca z odłączonymi encjami


### Attach()
Metoda *Attach()* przyłącza odłączony graf encji i zaczyna go śledzić.

Metoda *Attach()* ustawia główną encję na stan Added niezależnie od tego, czy posiada wartość klucza. Jeśli encje dzieci posiadają wartość klucza wówczas zaznaczane są jako *Unchanged*, a w przeciwnym razie jako *Added*.


``` csharp
context.Attach(entityGraph).State = state;
```

| Attach()  | Root entity with Key value  | Root Entity with Empty or CLR default value  | Child Entity with Key value  |  Child Entity with empty or CLR default value |
|---|---|---|---|---|
| EntityState.Added  | Added  | Added  | Unchanged  | Added  |
| EntityState.Modified  | Modified  | Exception  | Unchanged  | Added  |
| EntityState.Deleted  | Deleted  | Exception  | Unchanged  | Added  |



### Entry

``` csharp
context.Entry(order).State = EntityState.Modified
```

Wyrażenie przyłącza encję do kontekstu i ustawia stan na **Modified**. Ignoruje wszystkie pozostałe encje.

### Add()
Metody *DbContext.Add()* i *DbSet.Add()* przyłączają graf encji do kontekstu i ustawiają stan encji na **Added** niezależnie od tego czy posiadają wartość klucza czy też nie.

| Method | Root entity with/out Key value  | 	Root entity with/out Key |
|---|---|---|
| DbContext.Add  | Added  | Added  | 


### Update()

Metoda *Update()* przyłącza graf encji do kontekstu i ustawia stan poszczególnych encji zależnie od tego czy jest ustawiona wartość klucza.

| Update()  | Root entity with Key value  | Root Entity with Empty or CLR default value  | Child Entity with Key value  |  Child Entity with empty or CLR default value |
|---|---|---|---|---|
| DbContext.Update  | Modified  | Added  | Modified  | Added  |



### Delete()

Metoda *Delete()* ustawia stan głównej encji na **Deleted**.

| Delete()  | Root entity with Key value  | Root Entity with Empty or CLR default value  | Child Entity with Key value  |  Child Entity with empty or CLR default value |
|---|---|---|---|---|
| DbContext.Delete  | Deleted  | Exception  | Unchanged  | Added  |





## Change Tracker

Odczytanie stanu encji

~~~ csharp

Trace.WriteLine(context.Entry(customer).State);
foreach (var property in context.Entry(customer).Properties)
{
    Trace.WriteLine($"{property.Metadata.Name} {property.IsModified} {property.OriginalValue} -> {property.CurrentValue}");
}

~~~


## Surowy SQL

### Uruchomienie zapytania SQL i pobranie wyników

~~~ csharp
public IEnumerable<Customer> Get(string lastname)
{
    string sql = $"select * from dbo.customers where LastName = '{lastname}'";
    return context.Database.SqlQuery<Customer>(sql);
}
~~~

### Uruchomienie procedury składowanej
~~~ csharp
using (var context = new SampleContext())
{
    var books = context.Database
        .SqlQuery("GetAllCustomers")
        .ToList();
}
~~~

### Uruchomienie procedury składowanej z parametrami

~~~ csharp
using (var context = new SampleContext())
{
    var city = new SqlParameter("@City", "Warsaw");
    var customers = context
        .SqlQuery("GetCustomersByCity @City" , city)
        .ToList();
}
~~~


### Typy anonimowe i pobieranie wartości
~~~ csharp
var orderHeaders = db.Database.SqlQuery(
                    @"select c.Name as CustomerName, o.DateCreated, sum(oi.Price) as TotalPrice, 
                    count(oi.Price) as TotalItems
                    from  OrderItems  oi 
                    inner join Orders o on oi.OrderId = o.OrderId
                    inner join Customers c on o.CustomerId = c.CustomerId
                    group by oi.OrderId, c.Name, o.DateCreated");
~~~




## Indeksy

~~~ csharp

class EmployeeConfiguration : EntityTypeConfiguration<Employee>
{
    public EmployeeConfiguration()
    {

        HasIndex(p => p.Email)                
            .IsUnique();

    }
}

~~~

## Migracje


### Utworzenie triggera
1. Utwórz folder np. Scripts i plik OnDeleteOrderDetail.sql

~~~ sql
CREATE TRIGGER OnDeleteOrderDetail
    ON [dbo].[OrderDetails]
    FOR DELETE
AS
UPDATE [dbo].[Orders] SET ModifiedAt = getdate() WHERE Id = deleted.OrderId
~~~

2. Ustaw Build Action na Embedded Resource


3. Utwórz klasę migracji

~~~ csharp
public partial class AddTriggerOnDeleteOrderDetails : DbMigration
    {
        public override void Up()
        {
            SqlResource("MyApp.Scripts.201609301218380_AddTriggerOnDeleteOrderDetail_Up.sql", suppressTransaction: true);
        }
        
        public override void Down()
        {
            Sql("IF OBJECT_ID ('[OnDeleteOrderDetail]', 'TR') IS NOT NULL DROP TRIGGER OnDeleteOrderDetail");
        }
    }

~~~

### Utworzenie procedury składowanej

~~~ chsarp

public override void Up() 
{
  CreateStoredProcedure(
    "MyStoredProcedure",
    p => new
    {
        id = p.Int()
    },
    @"SELECT some-data FROM my-table WHERE id = @id"
  );
}

public override void Down() 
{
  DropStoredProcedure("MyStoredProcedure");
}

~~~


## Inicjalizatory

### Wbudowane inicjalizatory

| Typ | Opis |
|---|---|
| CreateDatabaseIfNotExists | utwórz bazę danych jeśli nie istnieje |
| DropCreateDatabaseAlways | zawsze usuń i utwórz bazę danych |
| DropCreateDatabaseIfModelChanges | usuń i utwórz bazę danych jeśli nastąpiły zmiany w modelu |


### Ustawienie inicjalizatora w kodzie

~~~ csharp
 public class MyContext : DbContext
 {
     public MyContext() : base("MyDbConnection")
     {
         Database.SetInitializer(new DropCreateDatabaseIfModelChanges<MyContext>());
     } 
 }
~~~

### Ustawienie inicjalizatora w pliku konfiguracyjnym

~~~ xml
<entityFramework>
        <contexts>
           <context type="MyApp.MyContext, MyContext">
           <databaseInitializer type="System.Data.Entity.DropCreateDatabaseAlways`1[[MyApp.MyContext, MyApp]], EntityFramework" />
           </context>
        </contexts>
    </entityFramework>
    
~~~

### Własny inicjalizator

~~~ csharp
public class MyDbInitializer : IDatabaseInitializer<MyContext>
    {
        public void MyDbInitializer(MyContext context)
        {
            if (!context.Database.Exists() || !context.Database.CompatibleWithModel(true))
            {
                context.Database.Delete();
                context.Database.Create();
            }
            // context.Database.ExecuteSqlCommand("Custom SQL Command here");
        }
    }
~~~



### Wyłączenie inicjalizatorów

~~~ csharp
public MyContext() : base("MyDbConnection")
{
    Database.SetInitializer<MyContext>(null);
 }
 ~~~
    
### Rozszerzenie wbudowanego inicjalizatora

Przykład wypełnienia danych

~~~ csharp
 public class MyDbInitializer : CreateDatabaseIfNotExists<MyContext>
    {
        private readonly CustomerFaker customerFaker;
        
        public MyDbInitializer(CustomerFaker customerFaker)
        {
            this.customerFaker = customerFaker;  
        }

        protected override void Seed(TransportContext context)
        {
            context.Customers.AddRange(customerFaker.Generate(1000));
         

            base.Seed(context);
        }

    }
~~~

## Śledzenie zapytań SQL


### Logowanie

~~~ csharp
public MyContext()
{
    this.Database.Log += Console.WriteLine;
}
~~~

### Formatowanie logów

Utworzenie własnego formattera
~~~ csharp

public class OneLineFormatter : DatabaseLogFormatter 
{ 
    public OneLineFormatter(DbContext context, Action<string> writeAction) 
        : base(context, writeAction) 
    { 
    } 
 
    public override void LogCommand<TResult>( 
        DbCommand command, DbCommandInterceptionContext<TResult> interceptionContext) 
    { 
        Write(string.Format( 
            "Context '{0}' is executing command '{1}'{2}", 
            Context.GetType().Name, 
            command.CommandText.Replace(Environment.NewLine, ""), 
            Environment.NewLine)); 
    } 
 
    public override void LogResult<TResult>( 
        DbCommand command, DbCommandInterceptionContext<TResult> interceptionContext) 
    { 
    } 
}
~~~

Rejestracja:

~~~ csharp
public class MyDbConfiguration : DbConfiguration 
{ 
    public MyDbConfiguration() 
    { 
        SetDatabaseLogFormatter((context, writeAction) => new OneLineFormatter(context, writeAction)); 
    } 
}
~~~


## Ładowanie powiązanych encji

### Zachłanne ładowanie (Eager Loading)

#### Ładowanie powiązanej właściwości

~~~ csharp
using (var context = new RentContext())
{
    var rentals = context.Vehicle
        .Include(p => p.Owner)                          
        .ToList();
}
~~~


#### Ładowanie zagnieżdżonych encji

~~~ csharp
using (var context = new RentContext())
{
    var rentals = context.Vehicle
        .Include(p => p.Owner).Select(b => b.Rentee).Include(b => b.Address);                               
        .ToList();
}
~~~

### Leniwe ładowanie (Lazy Loading)

MyContext.cs

~~~ csharp
public MyContext()
    : base("MyDbConnection")
{
    this.Configuration.LazyLoadingEnabled = true;
    this.Configuration.ProxyCreationEnabled = true;
}
~~~

Właściwości muszą być oznaczone jako **publiczne** i **wirtualne**. W przeciwnym razie Lazy Loading nie będzie działać!

~~~ csharp
public class Vehicle : Base
{
   public int Id { get; set; }

   public virtual Employee Owner { get; set; }

   public virtual ICollection<Employee> Passangers { get; set; }
}
~~~

### Jawne ładowanie (Explicit Loading)

#### Ładowanie powiązanej encji

~~~ csharp
context.Entry(vehicle).Reference(p => p.Owner).Load();
~~~

### Ładowanie powiązanej kolekcji
 
 ~~~ csharp
 context.Entry(vehicle).Collection(p => p.Passangers).Load();
~~~

### Filtrowanie ładowanej kolekcji

~~~ csharp
context.Entry(vehicle)
    .Collection(p => p.Passangers)
    .Query()
    .Where(p=>p.Gender = Gender.Female)
    .Load();
~~~

## Transakcje

### Transakcje bazy danych

~~~ csharp

private void Save(Order order)
{
    using (var context = new MyContext())
    using (var transaction = context.Database.BeginTransaction())
    {
        try
        {
            context.Orders.Add(order);
            context.SaveChanges();
            
            context.Customers.Add(order.Customer);
            context.SaveChanges();
            
            transaction.Commit();
        }
        catch(Exception)
        {
            transaction.Rollback();
        }
    }
}

~~~

### Rozproszone transakcje

Dodaj referencję do _System.Transactions_


~~~ csharp 

private static void Save(Order oder)
{
    using (var scope = new TransactionScope())
    {
        using (var context1 = new OrdersContext())
        {
            context1.Orders.Add(order);      
            context1.SaveChanges();
        }
 
        using (var context2 = new CustomersContext())
        {
            context2.Customers.Add(order.Customer);
            context2.SaveChanges();
        }
 
        scope.Complete();
    }
}
~~~

uwaga: w przypadku wykorzystania transakcji w metodzie asynchronicznej otrzymamy błąd. 
Dlatego należy dodać parametr w konstruktorze:

~~~ csharp
var transactionScope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled)
~~~
     

## Współbieżność

### Token

#### Konfiguracja za pomocą atrybutu

~~~ csharp
public class Employee
   {
       public int Id { get; set; }
 
       [ConcurrencyCheck]
       public string FirstName { get; set; }
       public string LastName { get; set; }
   }
~~~

#### Konfiguracja za pomocą FluentAPI

~~~ csharp
class EmployeeConfiguration : EntityTypeConfiguration<Employee>
    {
        public EmployeeConfiguration()
        {
            Property(p => p.FirstName)
                .IsConcurrencyToken();
        }
    }
~~~   
   
#### Wykrywanie kolizji

~~~ csharp
private static void ConcurencyTest()
   {
       using (var context = new MyContext())
       {
           var employee = context.Empoyees.Find(1);
           employee.FirstName = "John";

           bool saveFailed;
           do
           {
               saveFailed = false;

               try
               {
                   context.SaveChanges();
               }
               catch (DbUpdateConcurrencyException ex)
               {
                   saveFailed = true;

                   ex.Entries.Single().Reload();
               }

           } while (saveFailed);
       }
   }
   
~~~ 

### RowVersion

####  Konfiguracja za pomocą atrybutu

~~~ csharp
public class Employee
{
    [Timestamp]
    public byte[] RowVersion { get; set; }
}
~~~

####  Konfiguracja za pomocą FluentApi
   
~~~ csharp
public class Employee
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public byte[] RowVersion { get; set; }
}
~~~

~~~ csharp

class EmployeeConfiguration : EntityTypeConfiguration<Employee>
{
   public EmployeeConfiguration()
   {
       Property(p => p.RowVersion)
          .IsConcurrencyToken()
          .IsRowVersion();
   }
}

~~~

## Mapowanie operacji CRUD na procedury składowane

### Konfiguracja FluentApi
~~~ csharp
class EmployeeConfiguration : EntityTypeConfiguration<Employee>
    {
        public EmployeeConfiguration()
        {
            MapToStoredProcedures();
        }
    }
~~~

Utworzone zostaną procedury składowane.

### Modyfikacja nazw

Modyfikacja nazw procedur składowanych

~~~ csharp
class EmployeeConfiguration : EntityTypeConfiguration<Employee>
    {
        public EmployeeConfiguration()
        {
            MapToStoredProcedures(s =>
             {
                 s.Update(u => u.HasName("modify_employee"));
                 s.Delete(d => d.HasName("delete_employee"));
                 s.Insert(i => i.HasName("insert_employee"));
             });
        }
    }
~~~

## Odłączone encje

### Zapis odłączonej encji z użyciem biblioteki GraphDiff

Instalacja biblioteki
~~~ bash
PM> Install-Package RefactorThis.GraphDiff
~~~

~~~ csharp
private static void GraphDiffTest()
{
    Artist artist = new Artist { ArtistId = 1, FirstName = "The Artist Formerly Known as Prince" };

    using (var context = new MusicStoreContext())
    {
        context.UpdateGraph<Artist>(artist, map => map
               .OwnedCollection(p => p.Albums));

        context.SaveChanges();
    }
}
~~~

http://blog.brentmckendrick.com/introducing-graphdiff-for-entity-framework-code-first-allowing-automated-updates-of-a-graph-of-detached-entities/

