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
