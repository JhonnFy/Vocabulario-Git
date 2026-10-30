# Vocabulario y Convenciones del Proyecto

---

## Tipos de commits (Conventional Commits)

Convención estándar para nombrar los commits. El prefijo indica el **tipo de cambio** que se hizo. Facilita leer el historial y automatizar versiones.

| Prefijo | Uso | Ejemplo |
|---|---|---|
| `feat:` | **Nueva funcionalidad** | `feat: agregar endpoint de login` |
| `fix:` | **Corregir un error** | `fix: corregir cálculo de fecha en tickets` |
| `test:` | **Agregar o modificar pruebas** | `test: cubrir caso de email inválido` |
| `docs:` | **Documentación** | `docs: actualizar README de instalación` |
| `refactor:` | **Mejorar el código sin cambiar su comportamiento** | `refactor: extraer lógica de validación a un método` |

### Otros tipos comunes (opcionales)

| Prefijo | Uso |
|---|---|
| `style:` | Cambios de formato que no afectan la lógica (espacios, comas, indentación) |
| `chore:` | Tareas de mantenimiento (actualizar dependencias, configurar CI) |
| `perf:` | Mejoras de rendimiento |
| `build:` | Cambios en el sistema de build o dependencias |
| `ci:` | Cambios en la configuración de integración continua |

### Formato

```
<tipo>: <descripción corta en minúsculas>
```

**Ejemplos reales:**
```
feat: crear repositorio de usuarios con CRUD completo
fix: corregir null en ObtenerPorId cuando el id no existe
test: agregar pruebas de Update en CategoriaRepository
docs: documentar flujo de trabajo con Git
refactor: mover reglas de transformación a RulesServices
```

### Regla

> **El prefijo dice QUÉ se hizo. La descripción dice DÓNDE y POR QUÉ.**
> **Un commit = un cambio lógico. No mezclar `feat` con `fix` en el mismo commit.**

---

## Flujo de trabajo en equipo con Git y GitHub

### Paso a paso

**1. Clonar el repositorio (una sola vez por máquina)**
```bash
git clone https://github.com/tu-equipo/proyecto.git
cd proyecto
```

**2. Actualizar `main` antes de empezar**
```bash
git checkout main
git pull origin main
```
Siempre se parte de la última versión de `main`.

**3. Crear la rama de trabajo (sin pedir permiso)**
```bash
git checkout -b feature/mi-nueva-funcionalidad
```
La rama se crea en local. Convención de nombres: `feature/`, `fix/`, `hotfix/`.

**4. Trabajar y hacer commits en la rama**
```bash
git add .
git commit -m "Descripción clara del cambio"
```

**5. Subir la rama al remoto**
```bash
git push origin feature/mi-nueva-funcionalidad
```

**6. Abrir un Pull Request**
En GitHub aparece el botón "Compare & pull request". Ahí se avisa al equipo: "terminé, revisen".

**7. Revisión, aprobación y merge**
Alguien del equipo revisa, comenta o aprueba. Cuando está aprobado, se fusiona a `main`. La rama se borra después.

### Lo que NO se hace

- ❌ No se pide permiso al jefe para crear la rama. La crea uno mismo.
- ❌ No se trabaja directo sobre `main` (si la rama está protegida).
- ❌ No se avisa antes de empezar. Se avisa al final, con el Pull Request.

### La comunicación con el equipo

Es para coordinar qué está haciendo cada uno (evitar dos personas tocando lo mismo), no para pedir autorización para empezar.

---

## Protección de rama (`main`) — Reglas esenciales

Configuración en GitHub: **Settings → Branches → Branch protection rules**

### 1. Require a pull request before merging

**Qué hace:** prohíbe el `push` directo a `main`. Todo cambio debe entrar por un Pull Request.

**Por qué:** evita que alguien suba código sin revisión y rompa la rama principal.

### 2. Require approvals

**Qué hace:** exige un número mínimo de aprobaciones (1, 2, etc.) para que el PR se pueda fusionar.

**Regla clave de GitHub:** el autor del PR **no puede aprobar su propio PR**. La aprobación debe venir de otra persona.

**Por qué:** garantiza que al menos otra persona revisó el código antes de integrarlo.

### 3. Require status checks to pass before merging

**Qué hace:** el PR solo se fusiona si pasan las pruebas automáticas (tests, build, lint, etc.) configuradas en CI/CD (GitHub Actions u otro).

**Por qué:** evita fusionar código que rompe los tests o no compila.

### Orden recomendado para empezar

Activar en este orden, sin complicarse:

1. **Require a pull request before merging** (lo más básico)
2. **Require approvals** (mínimo 1)
3. **Require status checks** (cuando haya CI/CD configurado)

**No activar todo de golpe.** Demasiadas reglas al principio vuelven el flujo lento e insoportable.

---

## Inyección de dependencias (DI)

**Frase clave:**
> "Recibir lo que necesitas por constructor, en vez de crearlo adentro."

**Qué significa:**
La clase NO crea sus dependencias con `new`. Las recibe desde afuera (por el constructor) y las guarda en un campo privado `readonly`.

**Ejemplo del proyecto:**
- `Conexion(IConfiguration config)` → recibe la configuración.
- `CategoriaRepository(Conexion conexion)` → recibe la conexión.
- `CategoriaService(CategoriaRepository repo)` → recibe el repositorio.

**Por qué:**
- Desacopla: cada clase no sabe cómo se construye su dependencia.
- Testeable: puedes pasar un mock.
- Flexible: cambiar la implementación no rompe la clase.

**No es herencia ni polimorfismo.** Es otro concepto.

---

## readonly

**Qué es:** modificador de campo que garantiza que el valor solo se puede asignar **una vez**, en el constructor.

**Qué garantiza:**
- El constructor **sí** puede asignarlo.
- Después del constructor, **nadie** puede reasignarlo (ni métodos, ni propiedades).
- El campo queda fijo con ese objeto durante toda la vida del objeto.

**Dónde se usa en el proyecto:**
```csharp
private readonly Conexion _conexion;
private readonly IConfiguration _iConfiguration;
private readonly UsuarioRepository _repository;
```

**Por qué se usa:** los campos de dependencias (repositorio, conexión, configuración) **no deben cambiar** después de inyectarse. Si un método pudiera ponerlo en `null` o cambiarlo por otro, el objeto quedaría en un estado inconsistente.

**Regla:**
> **`readonly` en los campos de dependencias inyectadas = protección. Nadie los cambia después del constructor.**

**No es una llave sobre el constructor: es una llave sobre el CAMPO.** El constructor asigna una vez; el resto del código solo puede leerlo.

---

## static

**Qué es:** modificador de pertenencia. **No es modificador de acceso.**

**Qué hace:** indica que el método/propiedad/clase **pertenece a la clase**, no a los objetos creados de ella.

**Consecuencia:** se llama **desde la clase directamente**, **sin crear objeto**.

### Sin static

```csharp
public string ConvertirMayuscula(string cadena) { ... }

// Uso:
var r = new Rules();
r.ConvertirMayuscula("zul");
```

### Con static

```csharp
public static string ConvertirMayuscula(string cadena) { ... }

// Uso:
Rules.ConvertirMayuscula("zul");
```

### Diferencia con los modificadores de acceso

| Modificador | Qué cambia |
|---|---|
| `public` / `private` / `protected` / `internal` | **Quién** puede acceder |
| `static` | **Si necesita objeto** o no |

**Se combinan:** `public static` = todos pueden usarlo, sin crear objeto.

### Cuándo usarlo

- **Utilidades sin estado** (Rules, Math, helpers) → **`static`**
- **Clases con datos propios** (Ticket, Usuario) → **sin `static`**

---

## Capas de la arquitectura

| Capa | Qué hace |
|---|---|
| **Models** | Plantillas (datos) |
| **Data** | SQL y conexión |
| **Services** | Reglas de negocio |
| **Presentation** | Entrada/salida con el usuario |

### Cómo se implementa `Presentation` según la tecnología

| Tecnología | `Presentation` se implementa con... |
|---|---|
| Consola | `Menu.cs` (menú, `Console.ReadLine`) |
| Web API | `Controllers/` (`[HttpGet]`, JSON) |
| Web MVC | `Controllers/` + `Views/` |
| Escritorio | `Forms/` o `Views/` (botones, ventanas) |

**Siempre son 4 capas. Cambia la tecnología de `Presentation`, no la cantidad.**

---

## Reglas de arquitectura por capas

| Regla | Cómo se garantiza |
|---|---|
| `Program` solo llama a `Services` | Disciplina: nunca `new TicketRepository()` en `Program` |
| `Services` llama a `Data` | El servicio tiene el repositorio inyectado o lo instancia |
| Nadie más toca `Data` | Si otra capa lo toca, se rompe el flujo |

### Flujo de las operaciones

**Consultar (SELECT):**
```
Data → Services → Program
```

**Insertar / modificar / borrar:**
```
Program → Services → Data → MySQL
```

**El compilador no impide saltarse las reglas.** La garantía es de diseño y disciplina: `Program` nunca toca `Data` directamente.

---

## Estándar de nombres por capa

| Capa | Convención | Ejemplo |
| :--- | :--- | :--- |
| **Models** | Sin sufijo | `Categoria`, `Ticket`, `Usuario` |
| **Data (repositorios)** | Sufijo `Repository` | `CategoriaRepository`, `TicketRepository` |
| **Data (conexión)** | Descriptivo | `Conexion` |
| **Services** | Sufijo `Service` (singular) | `CategoriaService`, `TicketService` |
| **Presentation (consola)** | Descriptivo | `Menu` |
| **Presentation (web)** | Sufijo `Controller` | `TicketController` |

---

## Anatomía de un método en C#

```csharp
public bool Validar(Categoria c)
  ↑     ↑      ↑         ↑
  │     │      │         └── Parámetro (tipo + nombre)
  │     │      └──────────── Nombre del método
  │     └─────────────────── Tipo de retorno
  └───────────────────────── Modificador de acceso
```

### Partes de la firma

| Parte | Ejemplo | Qué significa |
|---|---|---|
| **Modificador de acceso** | `public` | Quién puede llamar al método (`public`, `private`, `protected`) |
| **Tipo de retorno** | `bool` | Qué devuelve el método al terminar |
| **Nombre del método** | `Validar` | Cómo se llama (PascalCase) |
| **Parámetro** | `Categoria c` | Tipo (`Categoria`) + nombre (`c`) del dato que recibe |

### Tipos de retorno comunes

| Retorno | Cuándo usarlo |
|---|---|
| `void` | No devuelve nada (guardar, imprimir) |
| `bool` | Devuelve sí/no (validar, existe) |
| `int` | Devuelve un número |
| `string` | Devuelve texto |
| `Categoria` / `Ticket` | Devuelve un objeto |
| `List<Ticket>` | Devuelve una lista de objetos |
