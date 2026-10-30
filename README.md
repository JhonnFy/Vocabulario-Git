# Vocabulario-Git
Vocabulario Git


## Descartar Cambios Locales
git restore NetTicket/NetTicket/Data/RepositoriesCRUD/CategoriaRepository.cs



## Protección de rama (`main`) — Reglas esenciales

Configuración en GitHub: **Settings → Branches → Branch protection rules**

### 1. Require a pull request before merging

**Qué hace:** prohíbe el `push` directo a `main`. Todo cambio debe entrar por un Pull Request.

**Por qué:** evita que alguien suba código sin revisión y rompa la rama principal.

---

### 2. Require approvals

**Qué hace:** exige un número mínimo de aprobaciones (1, 2, etc.) para que el PR se pueda fusionar.

**Regla clave de GitHub:** el autor del PR **no puede aprobar su propio PR**. La aprobación debe venir de otra persona.

**Por qué:** garantiza que al menos otra persona revisó el código antes de integrarlo.

---

### 3. Require status checks to pass before merging

**Qué hace:** el PR solo se fusiona si pasan las pruebas automáticas (tests, build, lint, etc.) configuradas en CI/CD (GitHub Actions u otro).

**Por qué:** evita fusionar código que rompe los tests o no compila.

---

### Orden recomendado para empezar

Activar en este orden, sin complicarse:

1. **Require a pull request before merging** (lo más básico)
2. **Require approvals** (mínimo 1)
3. **Require status checks** (cuando haya CI/CD configurado)

**No activar todo de golpe.** Demasiadas reglas al principio vuelven el flujo lento e insoportable.
