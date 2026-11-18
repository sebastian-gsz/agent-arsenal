---
name: managing-python-dependencies
description: |
  Garantiza una gestión correcta de las dependencies de Python, evitando el
  `pip install` global y respetando las herramientas propias de cada proyecto.

  Usa esta skill si se cumple alguna de las siguientes condiciones:
    1. Intentas ejecutar `pip install {package_name}`.
    2. Hay que añadir o modificar paquetes o dependencies de Python.
    3. Inicias un nuevo proyecto de Python.
    4. Creas un nuevo notebook, aunque sea solo con celdas de BigQuery.
    5. Generas código Python que incluye sentencias `import` de librerías de terceros.
    6. Antes de ejecutar scripts de Python por la terminal, para asegurarte de que el
       entorno virtual correcto está activo.
license: Apache-2.0
metadata:
  version: v1
  publisher: google
---

# Regla de gestión de dependencies de Python

> [!CAUTION]
>
> **ANTES de cualquier `pip install`**: DEBES detectar primero el dependency
> manager existente del proyecto y usarlo correctamente. NO sustituyas las
> herramientas ya establecidas del proyecto.

## Detección del dependency manager

Antes de instalar CUALQUIER paquete de Python, revisa en el workspace estos
archivos **en orden de prioridad**:

1.  **Señal:** `uv.lock` o `pyproject.toml` con `[tool.uv]`
    *   **Herramienta:** **uv**
    *   **Instalar:** `uv add <package>`
    *   **Configuración:** `uv sync`
2.  **Señal:** `pyproject.toml` con `[tool.poetry]`
    *   **Herramienta:** **Poetry**
    *   **Instalar:** `poetry add <package>`
    *   **Configuración:** `poetry install`
3.  **Señal:** `Pipfile`
    *   **Herramienta:** **Pipenv**
    *   **Instalar:** `pipenv install <package>`
    *   **Configuración:** `pipenv install`
4.  **Señal:** `environment.yml`
    *   **Herramienta:** **Conda**
    *   **Instalar:** `conda install <package>`
    *   **Configuración:** `conda env create -f environment.yml`
5.  **Señal:** solo `requirements.txt`
    *   **Herramienta:** **venv + pip**
    *   **Instalar:** `.venv/bin/pip install <package>`
    *   **Configuración:** `.venv/bin/pip install -r requirements.txt`
6.  **Señal:** ninguno de los anteriores
    *   **Herramienta:** **venv + pip** (predeterminado)
    *   **Instalar:** `.venv/bin/pip install <package>`
    *   **Configuración:** `.venv/bin/pip install -r requirements.txt`

## Predeterminado: venv + pip

Si no se detecta ningún dependency manager, usa **venv + pip +
requirements.txt** como opción predeterminada:

```bash
# Initialize environment
python3 -m venv .venv

# Add dependencies
.venv/bin/pip install <package>

# Preserve state
.venv/bin/pip freeze > requirements.txt
```

**Reglas para el flujo de trabajo con venv + pip:**

-   Usa siempre `.venv/bin/pip` o `.venv/bin/python` (ruta explícita).
-   Después de instalar, ejecuta: `.venv/bin/pip freeze > requirements.txt`.
-   Al configurar: `.venv/bin/pip install -r requirements.txt`.

## Prohibido

-   **NUNCA** ejecutes `pip install` globalmente
-   **NUNCA** sustituyas un dependency manager existente por otro distinto
