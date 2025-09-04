# Mi Primer Pipeline CI/CD: De Cero a Web con GitHub Actions

![GitHub Actions](https://github.githubassets.com/images/modules/site/actions/actions-illustration-2.svg)

Este repositorio es el resultado de un ejercicio práctico diseñado para enseñar los fundamentos de la Integración Continua y el Despliegue Continuo (CI/CD) utilizando Git, GitHub y GitHub Actions.

## 🎯 Objetivo del Ejercicio

El objetivo principal de este proyecto es construir un pipeline de CI/CD completamente automatizado. Este pipeline tomará el código de un sitio web simple desde este repositorio y lo desplegará automáticamente en la web pública a través de GitHub Pages cada vez que se detecte un nuevo cambio en la rama principal (`main`).

### ¿Qué se aprende y se practica?

Al completar este ejercicio, el estudiante habrá aprendido y practicado:
* **Fundamentos de CI/CD:** Entender el flujo de trabajo desde la escritura de código hasta su puesta en producción de forma automatizada.
* **Control de Versiones con Git:** Utilizar comandos esenciales de `git` para versionar el código, conectarse a un repositorio remoto y subir cambios.
* **Manejo de GitHub:** Crear y gestionar un repositorio, y comprender su interacción con las herramientas de CI/CD.
* **Automatización con GitHub Actions:** Escribir un workflow básico en formato YAML para definir los pasos del pipeline (checkout, build, deploy).
* **Seguridad y Autenticación:** Comprender por qué las contraseñas ya no se usan en Git por terminal y cómo generar y utilizar un **Personal Access Token (PAT)** de forma segura.
* **Gestión de Permisos (Scopes):** Entender que no todos los tokens son iguales y que es una buena práctica de seguridad otorgar solo los permisos necesarios (`repo` y `workflow`) para cada tarea.

---

## 🛠️ El Flujo de Trabajo (Pipeline) Explicado

Nuestro pipeline sigue tres fases fundamentales:

1.  **Disparo (Push):** Un desarrollador realiza un cambio en el código y lo sube al repositorio en la rama `main` usando el comando `git push`.
2.  **Integración Continua (CI):** GitHub Actions detecta este evento automáticamente. Inicia una máquina virtual, descarga la última versión del código y se prepara para el despliegue. En un proyecto real, en esta fase también se ejecutarían pruebas automatizadas para validar la calidad del código.
3.  **Despliegue Continuo (CD):** Una vez que la fase de CI es exitosa, el pipeline publica los archivos del sitio web en el entorno de producción de GitHub Pages, actualizando la página web en vivo.

---

## 📋 Guía de Implementación Paso a Paso

Aquí se detalla el orden lógico y completo para replicar este proyecto desde cero.

### Fase 1: Preparación en GitHub (Pasos Previos Indispensables)

Antes de escribir una sola línea de código, necesitamos configurar la autenticación y el lugar donde vivirá nuestro proyecto.

#### **Paso 1.1: Creación del Personal Access Token (PAT)**

**¿Por qué es necesario?** Por seguridad, GitHub ya no permite usar la contraseña de tu cuenta para autenticarte desde la terminal. Un PAT es una "contraseña" específica para aplicaciones, más segura porque puedes limitarle los permisos y revocarla en cualquier momento sin afectar tu cuenta principal.

**Instrucciones:**
1.  Ve a tu cuenta de GitHub → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)**.
2.  Haz clic en **"Generate new token (classic)"**.
3.  **Note:** Dale un nombre descriptivo, como `WSL Pipeline Access`.
4.  **Expiration:** Selecciona una duración (ej. `30 days`).
5.  **Select scopes (Permisos):** Aquí es donde otorgamos los permisos necesarios. Marca las siguientes dos casillas:
    * **`repo`:** Permite al token interactuar con el código de tus repositorios (clonar, subir, bajar cambios). Es **esencial** para el `git push`.
    * **`workflow`:** Permite al token modificar los archivos de GitHub Actions (los que están en la carpeta `.github/workflows`). Es **obligatorio** para poder subir tu archivo `deploy.yml`. Sin este permiso, GitHub rechazará el `push` que intente crear o cambiar un pipeline.
6.  Haz clic en **"Generate token"**.
7.  **¡ACCIÓN CRÍTICA!** Copia el token (`ghp_...`) y guárdalo en un lugar seguro y temporal. **Nunca más podrás volver a verlo**.

> ⚠️ **Advertencia de Seguridad:** NUNCA publiques tu Personal Access Token ni lo subas a un repositorio de GitHub. Trátalo como una contraseña. En este `README` y en tu código, nunca debe aparecer tu token real.

#### **Paso 1.2: Creación del Repositorio en GitHub**
1.  En GitHub, crea un nuevo repositorio público.
2.  Dale un nombre (ej. `mi-primer-pipeline`).
3.  **No** lo inicialices con un archivo `README` ni `.gitignore`. Lo haremos todo desde nuestra terminal.

---

### Fase 2: Configuración del Entorno Local (WSL Ubuntu)

Ahora pasamos a la terminal para empezar a trabajar.

1.  **Actualizar e Instalar Git:**
    ```bash
    sudo apt update && sudo apt upgrade -y
    sudo apt install git -y
    ```
2.  **Configurar tu identidad en Git:**
    ```bash
    git config --global user.name "Tu Nombre"
    git config --global user.email "tu_email@asociado_a_github.com"
    ```

---

### Fase 3: Creación del Proyecto y Conexión

1.  **Crear el proyecto localmente:**
    ```bash
    mkdir mi-primer-pipeline
    cd mi-primer-pipeline
    echo "<h1>Hola Mundo! Mi primer Pipeline funciona!</h1>" > index.html
    ```
2.  **Inicializar Git y conectar con GitHub:**
    ```bash
    git init -b main
    # Reemplaza la URL con la de tu repositorio
    git remote add origin [https://github.com/TU_USUARIO/mi-primer-pipeline.git](https://github.com/TU_USUARIO/mi-primer-pipeline.git)
    ```
3.  **Subir la versión inicial del código:**
    ```bash
    git add .
    git commit -m "Initial commit: Crear index.html"
    git push -u origin main
    ```
    > Al ejecutar `push`, te pedirá `Username` (tu usuario de GitHub) y `Password`. En la contraseña, **pega el Personal Access Token (PAT)** que generaste en el Paso 1.1.

---

### Fase 4: Construcción del Pipeline (GitHub Actions)

1.  **Crear la estructura de carpetas para el workflow:**
    ```bash
    mkdir -p .github/workflows
    ```
2.  **Crear el archivo de definición del pipeline** (puedes usar `nano` o `vi`):
    ```bash
    nano .github/workflows/deploy.yml
    ```
3.  **Pega el siguiente código YAML en el archivo:**
    ```yaml
    # Nombre del Pipeline que aparecerá en la pestaña "Actions" de GitHub
    name: Desplegar Mi Sitio Web

    # Disparador: Se ejecuta en cada 'push' a la rama 'main'
    on:
      push:
        branches:
          - main

    # Permisos necesarios para que el pipeline pueda interactuar con GitHub Pages
    permissions:
      contents: read
      pages: write
      id-token: write

    # Trabajos (Jobs) que se ejecutarán
    jobs:
      deploy:
        runs-on: ubuntu-latest
        steps:
          # 1. Descarga el código del repositorio a la máquina virtual
          - name: Checkout del código
            uses: actions/checkout@v4

          # 2. Configura el entorno para el despliegue a GitHub Pages
          - name: Configurar Pages
            uses: actions/configure-pages@v5

          # 3. "Construye" el artefacto (en este caso, empaqueta todo el directorio)
          - name: Construir el sitio
            uses: actions/upload-pages-artifact@v3
            with:
              path: '.'

          # 4. Despliega el artefacto en GitHub Pages
          - name: Desplegar en GitHub Pages
            id: deployment
            uses: actions/deploy-pages@v4
    ```

4.  **Subir la definición del pipeline al repositorio:**
    ```bash
    git add .
    git commit -m "CI: Añadir workflow de despliegue a GitHub Pages"
    git push
    ```
    > Este `push` es el que requiere el permiso `workflow` en tu PAT.

---

### Fase 5: Configuración Final y Verificación

1.  **Habilitar GitHub Pages (Paso manual único):**
    * En tu repositorio de GitHub, ve a **Settings** → **Pages**.
    * En la sección "Build and deployment", bajo "Source", selecciona **"GitHub Actions"**.
    * Este paso le dice a tu repositorio que debe esperar los despliegues desde un pipeline.

2.  **Verificar la ejecución del pipeline:**
    * Ve a la pestaña **"Actions"** de tu repositorio. Verás tu pipeline ejecutándose. Si falló la primera vez (porque el paso anterior no estaba hecho), puedes re-ejecutarlo desde esta misma pantalla con el botón "Re-run jobs".
    * Una vez que el pipeline muestre un ícono verde (✅ Success), tu sitio estará en línea.

3.  **¡Visita tu sitio web!**
    * La URL pública aparecerá en la sección **Settings** → **Pages**.

---

## ✅ Recomendaciones y Buenas Prácticas

* **Seguridad de Tokens:** Si tu token se filtra, ve inmediatamente a la configuración de GitHub y revócalo. Para proyectos más serios, utiliza **GitHub Secrets** para almacenar variables sensibles.
* **Ramificación (Branching):** En un entorno profesional, nunca se suben cambios directamente a la rama `main`. El flujo de trabajo correcto es:
    1.  Crear una nueva rama (ej. `feature/nuevo-titulo`).
    2.  Hacer los cambios y `push` a esa rama.
    3.  Crear un **Pull Request (PR)** en GitHub para fusionar tu rama con `main`.
    4.  El pipeline se puede configurar para que se ejecute cuando se aprueba el PR, asegurando que el código sea revisado antes de desplegarse.
* **Mensajes de Commit:** Escribe mensajes de commit claros y descriptivos. Ayudan a entender la historia del proyecto.
