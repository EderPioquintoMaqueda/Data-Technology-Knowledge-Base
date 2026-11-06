# 🌿 Git y GitHub | Version Control System

## ¿Qué es? | What is it?
**ES:** Git es un sistema de control de versiones distribuido. Permite rastrear cada cambio realizado en el código fuente, permitiendo regresar a versiones anteriores, experimentar con nuevas funciones sin romper el proyecto principal y colaborar con otros desarrolladores de forma eficiente. GitHub es la plataforma en la nube que aloja los repositorios de Git.

**EN:** Git is a distributed version control system. It allows tracking every change made to the source code, enabling you to revert to previous versions, experiment with new features without breaking the main project, and collaborate efficiently with other developers. GitHub is the cloud platform that hosts Git repositories.

---

## 🛠️ Flujo de Trabajo Fundamental | Fundamental Workflow

El ciclo de vida de un cambio en Git sigue este camino:

### 1. El Área de Trabajo (Working Directory)
Donde modificas tus archivos `.py`, `.sql` o `.md`.

### 2. El Área de Preparación (Staging Area / Index)
Marcas qué cambios quieres incluir en tu próxima "foto" del proyecto.
- **Comando**: `git add <archivo>`

### 3. El Repositorio Local (Local Repo)
Guardas la "foto" (commit) permanentemente en tu máquina.
- **Comando**: `git commit -m "Mensaje descriptivo"`

### 4. El Repositorio Remoto (Remote Repo / GitHub)
Subes tus cambios a la nube para respaldarlos o compartirlos.
- **Comando**: `git push origin main`

---

## 🚀 Conceptos Avanzados para Profesionales | Advanced Concepts

### 1. Ramas (Branches)
Permiten crear una línea de tiempo paralela.
- **Main/Master**: La versión estable y productiva.
- **Feature Branch**: Una rama para desarrollar una nueva funcionalidad sin riesgo.
- **Merge**: Fusionar la rama de la función con la rama principal una vez probada.

### 2. Pull Requests (PR)
En GitHub, es la solicitud formal para fusionar código. Permite que otros revisen tu trabajo, sugieran mejoras y aseguren la calidad antes de que el código llegue a la versión final.

### 3. Conflictos de Merge (Merge Conflicts)
Ocurren cuando dos personas modifican la misma línea del mismo archivo. Git no sabe cuál elegir y pide al humano que decCida la versión final.

---

## 💻 Guía de Comandos Rápidos | Quick Command Guide

| Acción | Comando | Descripción |
| :--- | :--- | :--- |
| **Iniciar Repo** | `git init` | Crea un repositorio nuevo en la carpeta actual. |
| **Clonar** | `git clone <url>` | Descarga un proyecto existente de GitHub. |
| **Estado** | `git status` | Muestra qué archivos han cambiado. |
| **Guardar** | `git add .` $\rightarrow$ `git commit -m "..."` | Prepara y guarda los cambios localmente. |
| **Sincronizar** | `git pull` | Descarga los cambios más recientes del servidor. |
| **Subir** | `git push` | Envía tus cambios locales al servidor. |
| **Ramificar** | `git checkout -b <nombre>` | Crea y salta a una nueva rama. |

---

## Aplicación Real en Datos | Real-world Data Application

**Caso: Colaboración en un Modelo de ML**
Un equipo de datos trabaja en la misma predicción de ventas:
1. El Analista A crea una rama `feature-random-forest`.
2. El Analista B crea una rama `feature-xgboost`.
3. Ambos suben sus cambios a GitHub.
4. El líder del equipo revisa ambos **Pull Requests**, compara las métricas de precisión y hace el `merge` del modelo que mejor funcionó.

---

## Conceptos Relacionados | Related Concepts
- [[02 - Programación]] $\longleftrightarrow$ Todo el código de Python debe vivir en un repositorio Git.
- [[08 - Ingeniería y Herramientas|Docker]] $\rightarrow$ Git guarda el código; Docker guarda el entorno donde ese código corre.

---
#git #github #devops #version-control
