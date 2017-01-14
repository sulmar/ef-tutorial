# Entity Framework


## Instalacja 

```
PM> Install-Package EntityFramework
```

## Podejścia do obsługi baz danych

### Database First 
- generowanie modelu na podstawie istniejącej bazy danych

### Model First
- generowanie modelu i bazy danych na podstawie diagramu

### Code First
- Generowanie  bazy danych na podstawie klas
- Generowanie modelu na podstawie istniejącej bazy danych

Utworzenie  modelu

``` csharp
public class Artist : Base
    {
        public int ArtistId { get; set; }
        
        public string FirstName { get; set; }
 
        public string LastName { get; set; }
    }

public enum AlbumType
{
	LP,
	SG
}

public class Album : Base
   {
       public int AlbumId { get; set; }
 
       public string Title { get; set; }
 
       public decimal Price { get; set; }
 
       public Artist Artist { get; set; }

       public AlbumType AlbumType { get; set; }

   }
```

Utworzenie kontekstu

``` csharp
public class MusicStoreContext : DbContext
   {
       public DbSet<Artist> Artists { get; set; }
       public DbSet<Album> Albums { get; set; }
 
       public MusicStoreContext()
           : base("MusicStoreConnection")
       {
           
       }
   }
```

## Inicjalizacja bazy danych

-	CreateDatabaseIfNotExists – domyślny inicjalizator. Jak nazwa wskazuje tworzy nową bazę danych jeśli nie istnieje we wskazanym miejscu w konfiguracji. Jeśli zmieni się model klas i uruchomimy aplikację wówczas będzie wygenerowany wyjątek.
-	DropCreateDatabaseIfModelChanges – jeśli zmieni się model klas inicjalizator usuwa istniejącą bazę i utworzy nową. Zatem nie musisz się martwić o zarządzanie strukturami bazy danych gdy twój model klas ulegnie zmianie, ale utracisz dane.
-	DropCreateDatabaseAlways – inicjalizator usuwa istniejącą bazę danych za każdym razem gdy uruchomisz aplikację, niezależnie od tego czy się zmienił model klas. To może być pomocne jeśli chcesz mieć świeżą bazę danych za każdym razem gdy uruchomiasz aplikację, na przykład podczas programowania.


``` csharp
public class MusicStoreContext: DbContext 
{
        
    public MusicStoreContext(): base("MusicStoreConnection") 
    {
        Database.SetInitializer<MusicStoreContext>(new CreateDatabaseIfNotExists<MusicStoreContext>());

        //Database.SetInitializer<MusicStoreContext>(new DropCreateDatabaseIfModelChanges<MusicStoreContext>());
        //Database.SetInitializer<MusicStoreContext>(new DropCreateDatabaseAlways<MusicStoreContext>());
        //Database.SetInitializer<MusicStoreContext>(new SchoolDBInitializer());
    }
    
}
``` 

-	Custom DB Initializer – własny inicjalizator dostosowany do twoich potrzeb poprzez dziedziczenie jednego z powyższych inicjalizatorów.

``` csharp
public class MyDBInitializer :  CreateDatabaseIfNotExists<MusicStoreContext>
{
    protected override void Seed(MusicStoreContext context)
    {
        base.Seed(context);
    }
} 
```

## Operacje CRUD

- Dodanie danych
- Pobranie danych
- Modyfikacja danych
- Usunięcie danych


## Monitoring

Wyświetlenie informacji o połączeniu

``` csharp
public MusicStoreContext()
{
    Console.WriteLine(this.Database.Connection.ConnectionString);
}
```

Śledzenie zapytań SQL

``` csharp
public MusicStoreContext()
{
    this.Database.Log = s => Console.WriteLine(s);
}
```

Formatowanie logów

``` csharp
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

public class MyDbConfiguration : DbConfiguration 
{ 
    public MyDbConfiguration() 
    { 
        SetDatabaseLogFormatter( 
            (context, writeAction) => new OneLineFormatter(context, writeAction)); 
    } 
}
```

## Aktualizacja bazy danych (migracje)

Włączenie migracji 
```
PM> Enable-Migrations
```

Dodanie migracji
```
PM> Add-Migration AddUrlToAlbum
```

Aktualizacja bazy danych
```
Update-Database
```

Wygenerowanie skryptu aktualizacji bazy danych
```
Update-Database -Script
```

Aktualizacja bazy danych do określonej migracji 
```
Update-Database -TargetMigration: AddUrlToAlbum
```

Aktualizacja bazy danych pomiędzy migracjami

```
Update-Database -SourceMigration: $InitialDatabase -TargetMigration: AddUrlToAlbum 
```

Uruchomienie migracji podczas uruchamiania aplikacji

``` csharp
Database.SetInitializer(new MigrateDatabaseToLatestVersion<MusicStoreContext, Configuration>());
```

Sprawdzenie czy istnieje bazy danych

``` csharp
context.Database.Exists();
```

Utworzenie procedury składowanej za pomocą migracji

``` sql
WITH    q AS
        (
        SELECT  FolderId
        FROM    dbo.Folders
        WHERE   FolderId = @FolderId
        UNION ALL
        SELECT  tc.FolderId
        FROM    q
        JOIN    dbo.Folders tc
        ON      tc.ParentId = q.FolderId
                
        )
DELETE
FROM    dbo.Folders
OUTPUT  DELETED.*
WHERE   EXISTS
        (
        SELECT  FolderId
        INTERSECT
        SELECT  FolderId
        FROM    q
        )
```


``` csharp
public partial class AddCascadeOnDeleteToFolder : DbMigration
   {
 
       public override void Up()
       {
           this.CreateStoredProcedure("dbo.uspDeleteFolder", 
               p => new { FolderId = p.Int() }, 
               Properties.Resources.DeleteFolder);
       }
       
       public override void Down()
       {
           this.DropStoredProcedure("dbo.uspDeleteFolder");
       }
   }
```


## Data Annotations

Konfiguracja pól za pomocą atrybutów

``` csharp
public class Artist : Base
    {
        [Key]
        public int ArtistId { get; set; }
 
        [MaxLength(50)]
        public string FirstName { get; set; }
 
        [Required]
        public string LastName { get; set; }
    }
```

Dostowanie nazwy tabeli

``` csharp
[Table("artists")]
    public class Artist : Base
    {
        public int ArtistId { get; set; }
        
        public string FirstName { get; set; }
 
        public string LastName { get; set; }
    }
```

## FluentAPI

Ustawienie schematu bazy danych

``` csharp

protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    modelBuilder.HasDefaultSchema("sales");
}

```

``` csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            modelBuilder.Entity<Artist>()
                .HasKey(p => p.ArtistId);
 
            modelBuilder.Entity<Artist>()
                .Property(p => p.FirstName)
                 .HasMaxLength(50);
 
            modelBuilder.Entity<Artist>()
                .Property(p => p.LastName)
                 .IsRequired();
 
             base.OnModelCreating(modelBuilder);
        }
```


Utworzenie klasy konfiguracyjnej

``` csharp
class ArtistConfiguration : EntityTypeConfiguration<Artist>
    {
        public ArtistConfiguration()
        {
            HasKey(p => p.ArtistId);
 
            Property(p => p.FirstName)
                .HasMaxLength(50);
 
            Property(p => p.LastName)
                .IsRequired();
 
        }
    }

protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        modelBuilder.Configurations.Add(new ArtistConfiguration());
        base.OnModelCreating(modelBuilder);
    }

```

Dostowanie nazwy tabeli

``` csharp
class ArtistConfiguration : EntityTypeConfiguration<Artist>
    {
        public ArtistConfiguration()
        {
            ToTable("artists");
        }
    }
```

## Inicjalizowanie bazy danych

Migrations/Configuration.cs

``` csharp
protected override void Seed(Altkom.Music.DAL.MusicStoreContext context)
       {
           
               context.Artists.AddOrUpdate(
                 p => p.LastName,
                 new Artist { LastName = "Prince" },
                 new Artist { LastName = "Jackson" }
                 
               );
           
       }

```


## Konwencje

### Standardowe Konwencje

 - Type Discovery
 - PrimaryKey Convention - właściwość ma w nazwie ID (wielkość liter nie znaczenia)
 - Relationship Convention
 - Complex Type Convention - tworzony jeśli klasa nie ma klucza
 - Connection String Convention

Usuwanie konwencji

``` csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    modelBuilder.Conventions.Remove<PluralizingTableNameConvention>();
}

``` 

Tworzenie własnych konwencji

``` csharp

public class DateTime2Convention : Convention
   {
      public DateTime2Convention()
        {
            this.Properties<DateTime>()
                 .Configure(c => c.HasColumnType("datetime2")
                     .HasPrecision(0));
       } 
    }

``` 

Przykład konwencji, która sprawdza nazwę właściwości

``` csharp
public class KeyConvention : Convention
    {
        public KeyConvention()
        {
            this.Properties<int>()
                 .Where(p => p.Name.EndsWith("Key"))
                 .Configure(p => p.IsKey());
        }
    }

``` 


## Relacje

https://msdn.microsoft.com/en-us/data/jj591620

## Mapowanie dziedziczenia klas


### TPH (Table Per Hierarchy)


### TPT (Table Per Type)
Tabele podklas zawierają tylko te kolumny, których właściwości nie są dziedziczone.
Podstawową zaletą jest to, że schemat bazy danych jest znormalizowany.

``` csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    modelBuilder.Entity<BankAccount>().ToTable("BankAccounts");
    modelBuilder.Entity<CreditCard>().ToTable("CreditCards");
}
``` 

### TPC (Table Per Concret Type)


## Linq

- Select
- Where
- OrderBy
- GroupBy
- ToList
- First, FirstOrDefault
- Single, SingleOrDefault
- Count, Sum, Average
- Union, Except
- Any, All


LinqPad
https://www.linqpad.net/

## Diagnostyka
Express Profiler
https://expressprofiler.codeplex.com/


## Attach

``` csharp
            using (var context = new MusicStoreContext())
            {
                context.Albums.Attach(album);
 
                context.Entry<Album>(album).State = EntityState.Modified;
 
                context.SaveChanges();
            }
```
Porada: warto stosować w przypadku usług sieciowych

W przypadku EF 6.x wywołanie metody DbSet.Add() powoduje wyszukiwanie wszystkich referencji encji w Navigation Property. Jeśli jakieś encje zostaną znalezione i nie są jeszcze śledzone przez Context również są oznaczone jako added. 

DbSet.Attach() ma to samo działanie za wyjątkiem takim, że wszystkie encje oznaczone są jako unchaged.

W EF Core to zostało zmienione.
- główna encja (root) jest zawsze w żądanym stanie (added dla DbSet.Add i uchanged dla DbSet.Attach)
- dla encji, które zostaną odnalezione podczas rekursywnego przeszukiwania 
 w navigation property stosowanie jest następująca reguła:
	- jeśli primary key jest generowany przez bazę danych 

Na podst.
https://docs.microsoft.com/en-us/ef/efcore-and-ef6/porting/ensure-requirements

### Biblioteka GraphDiff
Przy pracy z odłączonymi encji polecam bibliotekę GraphDiff
http://blog.brentmckendrick.com/introducing-graphdiff-for-entity-framework-code-first-allowing-automated-updates-of-a-graph-of-detached-entities/


Zapis odłączonej encji

``` csharp
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

```

## Stan obiektu


Pobieranie stanu obiektu

``` csharp
                if (context.ChangeTracker.HasChanges())
                {
                    var entries = context.ChangeTracker.Entries();
 
                    foreach (var entry in entries)
                    {
                        Console.WriteLine($"Entity Name: {entry.Entity.GetType().FullName}");
                        Console.WriteLine($"Status: {entry.State}");
                    }
                }

```

Przykładowe wykorzystanie - zapisywanie daty utworzenia i modyfikacji encji

``` csharp

public abstract class Base
   {
       public DateTime CreatedDate { get; set; }
       public DateTime ModifiedDate { get; set; }
   }


public override int SaveChanges()
        {
 
            var entries = this.ChangeTracker.Entries();
 
            var added = this.ChangeTracker.Entries<Base>()
                .Where(e => e.State == EntityState.Added)
                .Select(e => e.Entity);
 
            var modified = this.ChangeTracker.Entries<Base>()
                .Where(e => e.State == EntityState.Modified)
                .Select(e => e.Entity);
 
            var now = DateTime.UtcNow;
 
            foreach (var entry in added)
            {
                entry.CreatedDate = now;
                entry.ModifiedDate = now;
            }
 
            foreach (var entry in modified)
            {
                entry.ModifiedDate = now;
            }
 
            return base.SaveChanges();
        }
```

## Uruchamianie SQL

Uruchomienie zapytania SQL (fire and forgot) 

``` csharp
private static void SQLTest()
        {
            var sql = "update Price = 99 from dbo.Albums where AlbumId=1";
 
            using (var context = new MusicStoreContext())
            {
                context.Database.ExecuteSqlCommand(sql);
            }
        }
```

Przekazywanie parametrów

``` csharp

var albumId = 100;

using (var context = new VidiaContext())
           {
               var albumIdParameter = new System.Data.SqlClient.SqlParameter("@AlbumId", albumId);
 
               await context.Database.ExecuteSqlCommand("update Price = 99 from dbo.Albums where AlbumId = @AlbumId", albumIdParameter);
           }

```

Uruchomienie procedury składowanej i przekazanie parametrów

``` csharp
using (var
 context = new VidiaContext())
           {
               var folderIdParameter = new System.Data.SqlClient.SqlParameter("@FolderId", folderId);
 
               await context.Database.ExecuteSqlCommandAsync("uspDeleteFolder @FolderId", folderIdParameter);
           }

```

Porada: treść zapytań SQL przenieś do zasobów (resources) aplikacji


Uruchomienie procedury składowanej z parametrami i zwrócenie wyników

``` csharp
using(var context = new DatabaseContext())
    {
            var clientIdParameter = new SqlParameter("@ClientId", 4);

            var result = context.Database
                .SqlQuery<ResultForCampaign>("GetResultsForCampaign @ClientId", clientIdParameter)
                .ToList();
    }

```


## Transakcje

### Transakcje SQL Server

``` csharp

private static void TransactionTest()
        {
            Artist artist1 = new Artist();
            Artist artist2 = new Artist();
 
            using (var context = new MusicStoreContext())
            using (var transaction = context.Database.BeginTransaction())
            {
                try
                {
                    context.Artists.Add(artist1);
                    context.Artists.Add(artist2);
                    transaction.Commit();
                }
                catch(Exception)
                {
                    transaction.Rollback();
                }
            }
        }
```

### Transakcje rozproszone

``` csharp

private static void TransactionScopeTest()
{
    Artist artist1 = new Artist();
    Artist artist2 = new Artist();
 
    using (var scope = new TransactionScope())
    {
        using (var context1 = new MusicStoreContext())
        {
            context1.Artists.Add(artist1);
            context1.Artists.Add(artist2);
 
            context1.SaveChanges();
        }
 
        using (var context2 = new MusicStoreContext())
        {
            context2.Artists.Add(artist1);
            context2.SaveChanges();
        }
 
        scope.Complete();
    }
}

```

wskazówka: dodaj referencję do System.Transactions


## Konkurencyjność

### Konfiguracja bez pola Timestamp

Dodanie właściwości do encji za pomocą atrybutu

``` csharp

public class Artist : Base
   {
       public int ArtistId { get; set; }
 
       [ConcurrencyCheck]
       public string FirstName { get; set; }
 
       public string LastName { get; set; }
   }

```

Dodanie właściwości do encji za pomocą FluentAPI

``` csharp
class ArtistConfiguration : EntityTypeConfiguration<Artist>
    {
        public ArtistConfiguration()
        {
            Property(p => p.FirstName)
                .IsConcurrencyToken();
        }
    }


private static void ConcurencyTest()
       {
           using (var context = new MusicStoreContext())
           {
               var artist = context.Artists.Find(1);
               artist.FirstName = "Prince";
 
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

```

### Konfiguracja z użyciem pola Timestamp


Dodanie właściwości do encji za pomocą atrybutu

``` csharp

public class Artist : Base
    {
        [Timestamp]
        public byte[] RowVersion { get; set; }
    }
``` 

Dodanie właściwości do encji za pomocą FluentAPI


``` csharp
class ArtistConfiguration : EntityTypeConfiguration<Artist>
   {
       public ArtistConfiguration()
       {
           Property(p => p.RowVersion)
               .IsConcurrencyToken();
       }
   }

```

Obsługa transakcji

``` csharp
private static void ConcurencyTest()
       {
           using (var context = new MusicStoreContext())
           {
               var album = context.Albums.Find(1);
               album.Title = "The EF Blog";
 
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
 
                       // Update the values of the entity that failed to save from the store 
                       ex.Entries.Single().Reload();
                   }
 
               } while (saveFailed);
           }
       }

```

## Mapowanie operacji CRUD na procedury składowane


Mapowanie na procedury składowane

``` csharp
class ArtistConfiguration : EntityTypeConfiguration<Artist>
    {
        public ArtistConfiguration()
        {
            MapToStoredProcedures();
        }
    }

```


Modyfikacja nazw procedur składowanych


``` csharp
class ArtistConfiguration : EntityTypeConfiguration<Artist>
    {
        public ArtistConfiguration()
        {
            MapToStoredProcedures(s =>
             {
                 s.Update(u => u.HasName("modify_artist"));
                 s.Delete(d => d.HasName("delete_artist"));
                 s.Insert(i => i.HasName("insert_artist"));
             });
        }
    }
```

## Operacje asynchroniczne


Wyszukiwanie i zapis asynchroniczny

``` csharp
private static async Task AddAlbumAsyncTest()
        {
            using (var context = new MusicStoreContext())
            {
                var album = await context.Albums.FindAsync(1);
 
                album.Price = 39.99m;
 
                await context.SaveChangesAsync();
            }
        }
```


Pobieranie listy asynchronicznie

``` csharp
private static async Task GetAlbumsAsyncTest()
       {
           using (var context = new MusicStoreContext())
           {
               var albums = await context.Albums.ToListAsync();
 
               foreach (var album in albums)
               {
                   Console.WriteLine(album.Title);
               }
           }
       }

```

## Ładowanie powiązanych obiektów

### Ładowanie zachłanne (Eagerly Loading)

``` csharp
private static void EagerlyLoadingTest()
        {
            using (var context = new MusicStoreContext())
            {
                var albums = context.Albums
                    .Include(p=>p.Artist)
                    .Where(p => p.Price > 10)
                    .ToList();
            }
        }

```

### Ładowanie leniwe (Lazy Loading)

Należy włączyć w konfiguracji Lazy Loading i tworzenie proxy

``` csharp
public MusicStoreContext()
    : base("MusicStoreConnection")
{
    this.Configuration.LazyLoadingEnabled = true;
    this.Configuration.ProxyCreationEnabled = true;
}
```

Navigation Property musi być zdefiniowane jako publiczne i wirtualne

``` csharp
public class Artist : Base
   {
       public int ArtistId { get; set; }
 
       public string FirstName { get; set; }
 
       public string LastName { get; set; }
 
       public virtual ICollection<Album> Albums { get; set; }
   }
```

Wyłączenie Lazy Loading

``` csharp
public MusicStoreContext()
            : base("MusicStoreConnection")
        {
            this.Configuration.LazyLoadingEnabled = false;
        }
```

Porada: wyłącz Lazy Loading gdy używasz serializacji na przykład podczas pracy z Web Services.


## Optymalizacja

### Wyłączenie śledzenia
https://msdn.microsoft.com/en-us/data/jj556203

``` csharp
private static void NoTrackingTest()
        {
            using (var context = new MusicStoreContext())
            {
                var album = context.Albums
                    .Where(p => p.Price > 10)
                    .AsNoTracking()
                    .ToList();
            }
        }
```

Wyłączenie automatycznego wykrywania zmian
https://msdn.microsoft.com/en-us/data/jj556205

``` csharp
private static void DisableAutoDetectChanges()
        {
            using (var context = new MusicStoreContext())
            {
                try
                {
                    context.Configuration.AutoDetectChangesEnabled = false;
 
                    foreach (var album in context.Albums)
                    {
                        album.Price = 1.0m;
                    }
                }
                finally
                {
                    context.Configuration.AutoDetectChangesEnabled = true;
                }
 
                context.ChangeTracker.DetectChanges();
            }
        }  
```



### Generowanie widoków za pomocą Power Tools
https://msdn.microsoft.com/en-us/data/dn469601#powerTools

uwaga: po każdej zmianie modelu należy wygenerować widoki


Generowanie widoków z kodu
https://msdn.microsoft.com/en-us/data/dn469601.aspx#generating

### Automatyczne generowanie widoków

Instalacja biblioteki EFInteractiveViews

```
PM> Install-Package EFInteractiveViews
``` 

Użycie biblioteki

``` csharp
using (var ctx = new MyContext())
{
    InteractiveViews
        .SetViewCacheFactory(
            ctx, 
            new FileViewCacheFactory(@"C:\MyViews.xml"));
}

```

https://blog.3d-logic.com/2013/12/14/using-pre-generated-views-without-having-to-pre-generate-views-ef6/


### Cache'owanie danych

Instalacja biblioteki EFCache

```
PM> Install-Package EntityFramework.Cache
```

``` csharp
public class Configuration : DbConfiguration
    {
        public Configuration()
        {
            var transactionHandler = new CacheTransactionHandler(new InMemoryCache());
 
            AddInterceptor(transactionHandler);
 
            var cachingPolicy = new CachingPolicy();
 
            Loaded +=
              (sender, args) => args.ReplaceService<DbProviderServices>(
                (s, _) => new CachingProviderServices(s, transactionHandler,
                  cachingPolicy));
        }
    }

```

Uwaga: aby konfiguracja była widoczna dla EF klasa musi spełniać warunki:
•	Umieszczona w tym samym assembly co DbContext
•	Musu posiadać konstruktor bez parametrów
•	Musi być publiczna


## Metadane
Wygenerowanie dokumentacji

``` csharp
private static void GetDocumentationTest()
       {
           using (var context = new MusicStoreContext())
           {
               ObjectContext objContext = ((IObjectContextAdapter)context).ObjectContext;
               MetadataWorkspace workspace = objContext.MetadataWorkspace;
               IEnumerable<EntityType> tables = workspace.GetItems<EntityType>(DataSpace.SSpace);
 
               foreach (var table in tables)
               {
                   Console.WriteLine(table.Name);
                   Console.WriteLine("=========");
 
                   
                   foreach(var property in table.Properties)
                   {
                       var isPrimaryKey = table.KeyProperties.Contains(property);
                       
                       if (isPrimaryKey)
                           Console.Write("PK ");
 
                       Console.WriteLine($"{property.Name}");
 
                   }
               }
 
           }
```

## Spatial types

Dodaj referencje System.Data.Entity

### Typ geograficzny

``` csharp

public class Customer
    {
        public int CustomerId { get; set; }
 
        public string City { get; set; }
 
        public DbGeography Location { get; set; }
    }


private static void AddCustomersTest()
        {
            var customers = new List<Customer>
            {
                new Customer
                {
                     City = "Warszawa",
                     Location = DbGeography.FromText("POINT(52.13 21.0)"),
                },
 
                new Customer
                {
                     City = "Gdansk",
                     Location = DbGeography.FromText("POINT(54.22 18.40)"),
                },
 
                 new Customer
                {
                     City = "Poznań",
                     Location = DbGeography.FromText("POINT(52.25 16.55)"),
                },
 
 
                new Customer
                {
                     City = "Bydgoszcz",
                     Location = DbGeography.FromText("POINT(53.10 18.00)"),
                },
             };
 
            using (var context = new CustomersContext())
            {
                context.Customers.AddRange(customers);
 
                context.SaveChanges();
            }
        }




``` 

Wyszukiwanie najbliższego punktu

``` csharp

private static void FindClosestCustomerTest()
        {
            var myLocation = DbGeography.FromText("POINT(54.35 18.33)");
 
            using (var context = new CustomersContext())
            {
 
                var customer = (from u in context.Customers
                                orderby u.Location.Distance(myLocation)
                                select u).FirstOrDefault();
 
                Console.WriteLine($"The closest customer to you is: {customer.City}.");
 
            }
        }
```



### Typ geometryczny



## Oracle

Entity Framework Code First and Code First Migrations for Oracle Database
http://www.oracle.com/webfolder/technetwork/tutorials/obe/db/dotnet/CodeFirst/index.html

Oracle Developer Tools for Visual Studio 2015
http://www.oracle.com/technetwork/topics/dotnet/downloads/odacmsidownload-2745497.html


