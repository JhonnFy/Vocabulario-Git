# Contenedor de dependencias (DI Container)
Contenedor de dependencias
Dependency Injection Container

## ¿Qué es?

Un **contenedor de dependencias** es una herramienta que se encarga de **crear y entregar los objetos** que una clase necesita, sin que la clase tenga que construirlos ella misma.

En lugar de escribir a mano:

```csharp
var conexion = new Conexion(config);
var repository = new CategoriaRepository(conexion);
var service = new CategoriaService(repository);
var menu = new CategoriaMenu(service);
```

El contenedor lo hace por ti: tú le dices **qué clases existen** y **qué dependencias tienen**, y él las arma cuando se las pides.

---

## ¿Por qué usarlo?

- **Evita constructores gigantes:** una clase con 5 dependencias no tiene que recibir 5 parámetros manualmente en `Program`.
- **Centraliza la configuración:** en un solo lugar (típicamente `Program.cs`) se registra todo.
- **Facilita el testing:** puedes reemplazar implementaciones reales por mocks.
- **Es el estándar en .NET:** ASP.NET Core lo usa por defecto. Aprenderlo ahora te prepara para Web API.

---

## ¿Qué hay que instalar?

En una aplicación de consola .NET, el paquete NuGet:

```bash
dotnet add package Microsoft.Extensions.DependencyInjection
```

En Visual Studio: **clic derecho en el proyecto → Manage NuGet Packages → buscar `Microsoft.Extensions.DependencyInjection` → Install**.

---

## ¿Cómo se conecta con otras clases?

El flujo tiene 3 pasos:

### 1. Registrar los servicios

En `Program.cs`, se le dice al contenedor qué clases existen y cómo se construyen:

```csharp
var services = new ServiceCollection();

services.AddSingleton<IConfiguration>(configuration);
services.AddTransient<Conexion>();
services.AddTransient<CategoriaRepository>();
services.AddTransient<CategoriaService>();
services.AddTransient<CategoriaMenu>();
services.AddTransient<Menu>();

var provider = services.BuildServiceProvider();
```

### 2. Resolver (pedir) el objeto raíz

```csharp
var menu = provider.GetRequiredService<Menu>();
menu.Iniciar();
```

El contenedor mira el constructor de `Menu`, ve que necesita `CategoriaMenu`, y a su vez este necesita `CategoriaService`, que necesita `CategoriaRepository`, que necesita `Conexion`... **y construye toda la cadena automáticamente.**

### 3. Las clases no cambian

**Los constructores siguen igual.** La inyección por constructor no cambia:

```csharp
public class CategoriaService
{
    private readonly CategoriaRepository _repository;

    public CategoriaService(CategoriaRepository repository)
    {
        _repository = repository;
    }
}
```

El contenedor es el que se encarga de pasarle el `CategoriaRepository`. La clase no sabe que existe un contenedor.

---

## Tiempos de vida (lifetimes)

Al registrar un servicio, se elige cuánto vive:

| Lifetime | Qué hace | Cuándo usarlo |
|---|---|---|
| **`AddSingleton`** | Una sola instancia para toda la app | Configuración, conexiones compartidas |
| **`AddScoped`** | Una instancia por scope (petición web) | Servicios por request (en Web API) |
| **`AddTransient`** | Una instancia nueva cada vez que se pide | La mayoría de servicios y repositorios |

**Para la app de consola:** `AddTransient` para servicios y repositorios, `AddSingleton` para `IConfiguration`.

---

## Lo que hay que tener en cuenta

- **El contenedor no reemplaza la inyección por constructor:** la complementa. Las clases siguen recibiendo sus dependencias por constructor.
- **No todo va al contenedor:** solo las clases que tienen dependencias que construir. Las clases estáticas (`RulesServices`) no se registran.
- **`Program.cs` se vuelve el “compositor”:** ahí se registra todo, y ahí se pide el objeto raíz (`Menu`).
- **No abuses del contenedor:** si una clase tiene 10 dependencias, el problema no es el contenedor: es que la clase hace demasiado.
- **En consola, el contenedor es opcional.** Puedes seguir con `new` manual. Pero en Web API es obligatorio.
- **El contenedor construye en orden:** si `A` necesita `B` y `B` necesita `C`, el contenedor crea `C`, luego `B`, luego `A`. No tienes que preocuparte por el orden de creación.

---

## Ejemplo de la diferencia

**Sin contenedor:**

```csharp
var config = new ConfigurationBuilder().AddJsonFile("appsettings.json").Build();
var conexion = new Conexion(config);
var repo = new CategoriaRepository(conexion);
var service = new CategoriaService(repo);
var catMenu = new CategoriaMenu(service);
var menu = new Menu(catMenu);
menu.Iniciar();
```

**Con contenedor:**

```csharp
var services = new ServiceCollection();
services.AddSingleton<IConfiguration>(config);
services.AddTransient<Conexion>();
services.AddTransient<CategoriaRepository>();
services.AddTransient<CategoriaService>();
services.AddTransient<CategoriaMenu>();
services.AddTransient<Menu>();

var provider = services.BuildServiceProvider();
provider.GetRequiredService<Menu>().Iniciar();
```

**Mismo resultado. Menos código manual. Más escalable cuando crecen las dependencias.**
