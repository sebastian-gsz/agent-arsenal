# Dataflow Flex Template - Configuración de Imagen Docker Única

Este documento proporciona directrices de referencia para construir y lanzar
Dataflow Flex templates usando la configuración de **Imagen Docker Única**.

---

## 1. Directrices del Dockerfile

Usa esta plantilla para construir imágenes de contenedor personalizadas donde los
workers ejecutan la **exactamente la misma imagen** que lanzó el job.

### Plantilla

```dockerfile
ARG IMAGE_PYTHON_VERSION=3.11

# --- Stage 1: Get Python Flex Template Launcher Binary ---
FROM gcr.io/dataflow-templates-base/python3-template-launcher-base:latest AS launcher

# --- Stage 2: Build Application Image ---

# Always look up and use the latest Apache Beam SDK version unless user instructed to use a specific version.
FROM apache/beam_python${IMAGE_PYTHON_VERSION}_sdk:<LATEST_VERSION>

# 1. Copy Flex Template launcher binary (Path is STRICT)
COPY --from=launcher /opt/google/dataflow/python_template_launcher /opt/google/dataflow/python_template_launcher

# 2. Set working directory
WORKDIR /template

# 3. Install Python Dependencies
COPY requirements.txt setup.py ./
RUN pip install --no-cache-dir -r requirements.txt

# 4. Copy custom source FIRST
COPY src/ src/

# 5. Install Workspace Package
RUN pip install --no-cache-dir -e .

# 6. Specify Pipeline Entrypoint
ENV FLEX_TEMPLATE_PYTHON_PY_FILE=/template/<PATH_TO_PIPELINE_LAUNCHER_FILE>
```

> [!IMPORTANT]
> **REGLAS CRÍTICAS:**
> - NO sobrescribas el ENTRYPOINT (los workers necesitan /opt/apache/beam/boot).
> - NO establezcas FLEX_TEMPLATE_PYTHON_REQUIREMENTS_FILE ni
>   FLEX_TEMPLATE_PYTHON_SETUP_FILE
>   (Estas dependencias ya están instaladas en la imagen).
---

## 2. Guía de lanzamiento

*   **Pasa explícitamente la imagen de worker personalizada**:
    *   Si construyes una imagen personalizada para tu Flex Template,
        **debes suministrarla en runtime** al lanzar el template.
    *   **Ejemplo (con `gcloud`)**: pásala como cadena de parámetro:
        `--parameters sdk_container_image="[IMAGE_URI]"`
