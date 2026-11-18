---
name: gcp-pipeline-resource-provisioning
description: |
  Automatiza la creación y el aprovisionamiento declarativo de recursos para pipelines de datos, soportando BigQuery, Dataform, Dataproc, BigQuery Data Transfer Service (DTS) y otros recursos. Gestiona configuraciones por entorno (dev, staging, prod) mediante un archivo deployment.yaml.
  Úsala cuando:
  - Se modifique o cree deployment.yaml para la configuración de despliegue.
  - Se resuelvan variables específicas del entorno (p. ej., IDs de proyecto, regiones) para el despliegue.
  - Se aprovisione infraestructura soportada como datasets/tablas de BigQuery, recursos de Dataform o recursos de DTS vía deployment.yaml.
  No la uses cuando:
  - Los recursos ya existan.
  - Se gestionen recursos no soportados por `gcloud beta orchestration-pipelines resource-types list`.
  - Se gestione infraestructura general de nube (VMs, redes, Kubernetes, políticas IAM), para lo que es más adecuado Terraform.
  - La infraestructura abarque varios proveedores de nube (AWS, Azure, etc.).
  - Ya se use Terraform para los recursos objetivo.
license: Apache-2.0
metadata:
  version: v1
  publisher: google
---

## Cómo usar esta skill

Crea o actualiza el archivo `deployment.yaml` existente y despliega los
recursos. Todos los archivos de configuración DEBEN mantenerse juntos en la
raíz del repositorio.

### Etiquetas obligatorias

> [!IMPORTANT]
>
> Cada vez que generes definiciones de recursos en `deployment.yaml`, **DEBES**
> rellenar directamente la etiqueta `datacloud` bajo `definition.labels` de
> todos los recursos para rastrear el origen de su creación. Determina el valor
> según tu entorno de IDE actual:
>
> -   En Antigravity, asigna `datacloud: "antigravity"`
> -   En VS Code, asigna `datacloud: "vscode"`
> -   En cualquier otro entorno, asigna `datacloud: "other"`
>
> No uses una sustitución de variable para esta etiqueta; incrusta el valor de
> cadena apropiado directamente en cada definición de recurso (p. ej.,
> sustituyendo los placeholders `__REQUIRED_LABEL__`).
>
> **Regla especial para la ingesta de BigQuery DTS:** cada vez que generes un
> `bigquerydatatransfer.transferConfig` en `deployment.yaml`, **DEBES** definir
> también explícitamente su `bigquery.dataset` de destino en el mismo archivo y
> aplicarle la etiqueta `datacloud`. Hazlo incluso si el dataset ya existe, para
> garantizar que las etiquetas del dataset de destino se parchean y actualizan.

### Paso 1: Tipos de recursos soportados

El framework soporta el despliegue de diversos recursos de GCP. Para ver la
lista completa de tipos de recursos soportados, ejecuta el siguiente comando:

```
gcloud beta orchestration-pipelines resource-types list
```

Consulta: references/gcp-pipeline-resource-provisioning_spec.md para entender
la plantilla de `deployment.yaml`.

### Paso 2: Descubrir los parámetros del entorno

Antes de generar configuraciones, descubre los valores reales del proyecto,
la región, el entorno y el commit SHA de destino.

> [!TIP]
>
> Si `deployment.yaml` ya existe en la raíz del repositorio, prioriza extraer
> `project` y `region` de la configuración del entorno de destino
> (p. ej., `dev`).

1.  **ID de proyecto**:

    ```bash
    gcloud config get project
    ```

2.  **Número de proyecto**:

    ```bash
    gcloud projects describe $(gcloud config get project) --format="value(projectNumber)"
    ```

3.  **Región**:

    ```bash
    gcloud config get-value compute/region
    ```

4.  **Commit SHA**:

    ```bash
    git rev-parse HEAD
    ```

5.  **Nombre del entorno**: si hace falta inicializarlo, DEBES preguntar al
    usuario el nombre del entorno. Si no lo proporciona, usa **dev** por
    defecto.

> [!TIP]
>
> Usa estos comandos para sustituir placeholders como `YOUR_PROJECT_ID` por
> valores reales. Elimina siempre los comentarios TODO asociados una vez
> sustituidos.

### Paso 3: Generar o actualizar deployment.yaml

Crea o actualiza `deployment.yaml` en la raíz del repositorio. Este archivo
mapea los entornos soportados (**dev**, **stage**, **prod**) con sus
configuraciones y recursos específicos.

> [!TIP]
>
> **Usa la spec de referencia**: el agente puede usar el archivo
> **`references/gcp_pipeline_resource_provisioning_spec.md`** como plantilla.
> Incluye definiciones de ejemplo para algunos de los tipos de recursos
> soportados. Copia y adapta los bloques de recursos necesarios al
> `deployment.yaml`. Usa `gcloud beta orchestration-pipelines resource-types list`
> cuando sea necesario.

> [!IMPORTANT]
>
> **Gestión de secretos y privacidad (CRÍTICO)**: NUNCA incrustes secretos en
> texto plano en `deployment.yaml`.
>
> -   **Datos sensibles (secretos):** la información sensible, como contraseñas,
>     API keys y otros datos delicados, DEBE almacenarse en Secret Manager y
>     declararse en el bloque `secrets:` de `deployment.yaml`.
> -   **Datos no sensibles (variables):** la configuración general (p. ej.,
>     nombres de datasets, IDs de tablas, regiones) puede declararse en el
>     bloque `variables:`.
> -   **Sustitución vía `{{ VAR }}`:** tanto `variables:` como `secrets:` DEBEN
>     usarse como sustituciones `{{ VARIABLE_NAME }}` en las definiciones de
>     recursos.
> -   **Sin creación:** el agente NO DEBE usar el framework para *crear*
>     secretos nuevos. Si `gcloud` indica que el secreto no existe, el agente
>     DEBE pedir al usuario que lo cree manualmente y luego re-verificar.
> -   **Política de solo referencia:** el rol del agente se limita
>     estrictamente a *referenciar* secretos existentes. El agente NUNCA DEBE
>     leer, imprimir ni inspeccionar los valores de los secretos.
> -   **Despliegue seguro:** la inyección del valor real ocurre durante la
>     ejecución del despliegue. El agente solo aporta la referencia.
> -   **Gestión manual de secretos:** recomienda al usuario gestionar los
>     payloads y versiones de los secretos manualmente.

### Paso 4: Validación

El agente DEBE validar el `deployment.yaml` antes de generar el script de
despliegue. Esto garantiza que la configuración es sintácticamente correcta y
que todas las variables son resolubles.

```
gcloud beta orchestration-pipelines validate --environment=<ENV_NAME>
```

### Paso 5: Despliegue

Ejecuta el siguiente comando para desplegar los recursos en el entorno de
destino.

```
gcloud beta orchestration-pipelines deploy --environment=<ENV_NAME> --local
```

> [!NOTE]
>
> Si se está creando una transferencia nueva, asegúrate de NO eliminar el
> recurso de transferencia DTS de `deployment.yaml` después de que complete su
> ejecución.

## Definición de terminado

-   `deployment.yaml` existe en la raíz del repositorio con los valores reales
    descubiertos (sin placeholders) y las definiciones de recursos correctas.
-   El agente ejecuta el comando de despliegue para realizar el despliegue, y
    este se ejecuta con éxito (código de salida 0).
