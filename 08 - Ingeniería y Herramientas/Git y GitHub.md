# 🌿 Git y GitHub: Control de Versiones y Colaboración Profesional | Git & GitHub: Professional Version Control & Collaboration

**ES:** Para un programador novato, Git es una herramienta para "guardar copias". Para un ingeniero de datos, Git es la **única fuente de verdad** del proyecto. En equipos de alto rendimiento, el código no se "sube", se propone mediante un proceso de revisión técnica que asegura la estabilidad del sistema.

**EN:** For a novice programmer, Git is a tool for "saving copies". For a data engineer, Git is the **single source of truth** for the project. In high-performance teams, code is not simply "uploaded"; it is proposed through a technical review process that ensures system stability.

---

## 1. Professional Workflow: Git Flow | Flujo de Trabajo Profesional

El uso de una sola rama (`main`) es peligroso en entornos productivos. El estándar industrial es el **Git Flow**, que separa el desarrollo de la estabilidad.

### A. La Jerarquía de Ramas
- **`main` (o `master`):** Código sagrado. Solo contiene versiones que ya están en producción y funcionan perfectamente.
- **`develop`:** La rama de integración. Aquí se reúnen todas las nuevas funcionalidades antes de pasar a producción.
- **`feature/nombre-tarea`:** Ramas temporales. Cada nueva función o corrección vive aquí hasta que es aprobada.
- **`hotfix/error-critico`:** Ramas urgentes que saltan la cola para arreglar un fallo en producción inmediatamente.

### B. El Ciclo de Vida de un Cambio:
$\text{Feature Branch} \rightarrow \text{Commit} \rightarrow \text{Pull Request (PR)} \rightarrow \text{Code Review} \rightarrow \text{Merge a Develop} \rightarrow \text{Merge a Main}$.

---

## 2. The Art of the Commit: Conventional Commits | El Arte del Commit

Un mensaje como "fix bug" es inútil en un proyecto con miles de commits. El estándar **Conventional Commits** permite automatizar el historial del proyecto.

**Estructura:** `<tipo>(<alcance>): <descripción>`
- `feat:` Una nueva funcionalidad.
- `fix:` Corrección de un error.
- `docs:` Cambios en la documentación.
- `refactor:` Cambio de código que no añade función ni arregla bug (mejora de calidad).
- `perf:` Mejora de rendimiento.

**Ejemplo Pro:** `feat(auth): implement jwt token validation for api endpoints`

---

## 3. Automation with GitHub Actions (CI/CD) | Automatización con GitHub Actions

El control de versiones moderno es activo. Usamos **CI/CD (Continuous Integration / Continuous Deployment)** para que la máquina revise el código por nosotros.

### El Pipeline de Calidad:
Cuando abres un **Pull Request**, GitHub Actions dispara automáticamente:
1. **Linters:** Verifica que el código siga el estilo (Ej: `flake8` o `black` en Python).
2. **Unit Tests:** Ejecuta todos los tests (`pytest`) para asegurar que el nuevo cambio no rompió funciones existentes.
3. **Security Scan:** Busca secretos (como llaves de API) expuestos por error en el código.
**Resultado:** Si algún paso falla, el botón de "Merge" se bloquea.

---

## 4. DVC: Versioning Data and Models | DVC: Versionando Datos y Modelos

Git es excelente para texto, pero **terrible para archivos binarios grandes** (Datasets de 10GB, Pesos de LLMs de 50GB). Guardar estos archivos en Git ralentiza el repositorio hasta hacerlo inutilizable.

### La Solución: DVC (Data Version Control)
DVC actúa como una "capa" sobre Git.
- **Cómo funciona:** DVC guarda el archivo pesado en la nube (S3, Azure Blob, Google Drive) y guarda en Git solo un **archivo pequeño de puntero** (`.dvc`).
- **Flujo Experto:** 
  `git checkout v1.0` $\rightarrow$ `dvc checkout` $\rightarrow$ Ahora tienes el código v1.0 y el dataset exacto que se usó para entrenar ese modelo.

---

## 💻 Professional Git Commands Cheat Sheet

| Objetivo | Comando Pro | Explicación |
| :--- | :--- | :--- |
| **Sincronizar Limpio** | `git pull --rebase origin main` | Trae los cambios y pone tus commits arriba, evitando "merge commits" innecesarios. |
| **Limpieza de Historia** | `git commit --amend` | Corrige el último commit sin crear uno nuevo. |
| **Viaje en el Tiempo** | `git checkout <hash>` | Salta a un estado exacto del proyecto en el pasado. |
| **Guardado Temporal** | `git stash` $\rightarrow$ `git stash pop` | Guarda cambios no terminados para cambiar de rama rápidamente sin hacer commit. |
| **Cirugía de Commits** | `git rebase -i HEAD~3` | Permite editar, borrar o fusionar los últimos 3 commits antes de subirlos. |

#git #github #devops #cicd #dvc #version-control #software-engineering
