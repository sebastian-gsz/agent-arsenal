---
name: federate-lakehouse-catalog
description: 'Configura catálogos federados de Google Cloud Lakehouse hacia Iceberg
  REST Catalogs remotos. Catálogos admitidos actualmente: Databricks Unity, AWS Glue.
  Nubes que alojan esos catálogos: GCP, AWS. El caso de uso principal es conectarse
  a datos remotos para consultarlos desde motores de GCP (BigQuery, Spark). Ejemplos
  de cuándo usar esta skill: "federar mi lakehouse catalog a databricks", "consultar
  datos en databricks", "consultar datos en s3", "conectarme a aws glue". NO usar
  para ejecución SQL directa en bases de datos remotas (p. ej., Databricks SQL) ni
  para administrar clústeres e infraestructura remotos (p. ej., clústeres de
  Databricks, jobs de AWS Glue).'
license: Apache-2.0
metadata:
  version: v1
  publisher: google
---

# Federar el Lakehouse Catalog mediante Cross-cloud Lakehouse

Esta skill describe cómo configurar un catálogo federado en BigQuery para
consultar datos de catálogos remotos como Databricks Unity Catalog o AWS Glue
Data Catalog en AWS a través de la internet pública.

## Requisitos previos

-   Para Databricks: URL del Databricks Workspace y OAuth Service Principal
    (Client ID y Secret) con acceso de lectura.
-   Para AWS Glue: acceso de administrador de AWS para crear roles IAM y
    políticas de permisos.
-   Proyecto de Google Cloud activo con acceso administrativo para crear
    recursos de lakehouse y, en el caso de Databricks, secrets.

## Procedimiento

### Paso 1: Recopilación de información y selección de región

Antes de ejecutar cualquier comando, el agente **DEBE** recopilar del usuario
la siguiente información:

1.  Determina a qué catálogo quiere federar el usuario (p. ej., Databricks
    Unity o AWS Glue) y verifica que esté soportado.
2.  Determina dónde se encuentran los datos remotos (la región específica de
    AWS).
3.  Usando la buena práctica de emparejamiento de regiones de la sección de
    Gotchas, ayuda al usuario a elegir la región de GCP óptima para minimizar
    la latencia.
4.  Recopila las variables de configuración necesarias para el flujo elegido
    (p. ej., credenciales de Databricks o el AWS Account ID).

Continúa con los siguientes pasos solo cuando esta información esté confirmada.

### Paso 2: Verificación de las API

Verifica que las API de Google Cloud requeridas estén habilitadas para el
proyecto:

```bash
gcloud services check biglake.googleapis.com
```

Si la API no está habilitada, pide explícitamente permiso al usuario para
habilitarla. NO continúes sin su confirmación.

### Flujo A: Databricks Unity Catalog

#### 1. Crear un secret regional para las credenciales

Guarda el client ID y el secret de Databricks en Secret Manager. Asegúrate de
que la API `secretmanager.googleapis.com` esté habilitada. El secret **DEBE**
estar en la misma región que tu Lakehouse catalog.

1.  Crea un archivo JSON llamado `credentials.json`:

```json
{
  "client_id": "<CLIENT_ID>",
  "client_secret": "<CLIENT_SECRET>"
}
```

1.  Configura el override del endpoint de la API de Secret Manager para la
    región:

```bash
gcloud config set api_endpoint_overrides/secretmanager https://secretmanager.<REGION>.rep.googleapis.com/
```

1.  Crea el secret:

```bash
gcloud secrets create <SECRET_NAME> \
  --location="<REGION>" \
  --project="<PROJECT_ID>" \
  --data-file=credentials.json
```

#### 2. Crear el catálogo federado

Crea un catálogo BigLake Iceberg de tipo `federated` que apunte a Databricks.

```bash
gcloud alpha biglake iceberg catalogs create <CATALOG_NAME> \
   --project="<PROJECT_ID>" \
   --primary-location="<REGION>" \
   --catalog-type="federated" \
   --federated-catalog-type="unity" \
   --secret-name="projects/<PROJECT_ID>/locations/<REGION>/secrets/<SECRET_NAME>" \
   --unity-instance-name="<UNITY_INSTANCE_NAME>" \
   --unity-catalog-name="<UNITY_CATALOG_NAME>" \
   --refresh-interval="300s"
```

#### 3. Otorgar al catálogo acceso al secret

Otorga a la cuenta de servicio creada para el catálogo permiso para leer el
secret.

1.  Obtén el email de la cuenta de servicio describiendo el catálogo:

```bash
gcloud alpha biglake iceberg catalogs describe <CATALOG_NAME> \
    --project="<PROJECT_ID>" \
    --location="<REGION>" \
    --format="value(biglake-service-account-id)"
```

1.  Otorga el acceso:

```bash
gcloud secrets add-iam-policy-binding <SECRET_NAME> \
  --project="<PROJECT_ID>" \
  --location="<REGION>" \
  --member="serviceAccount:<SERVICE_ACCOUNT_EMAIL>" \
  --role="roles/secretmanager.secretAccessor"
```

### Flujo B: AWS Glue

#### 1. Crear el rol IAM de AWS con una trust policy de placeholder

Lakehouse aprovisiona un ID de cuenta de servicio de Google después de crear el
catálogo. Crea primero el rol IAM de AWS con una trust policy de placeholder.

1.  Crea un archivo llamado `trust_policy.json`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "accounts.google.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "accounts.google.com:aud": ["PLACEHOLDER_VALUE"],
          "accounts.google.com:sub": ["PLACEHOLDER_VALUE"]
        }
      }
    }
  ]
}
```

1.  Ejecuta el comando de AWS CLI para crear el rol:

```bash
aws iam create-role \
  --role-name <AWS_ROLE_NAME> \
  --assume-role-policy-document file://trust_policy.json \
  --max-session-duration 43200
```

#### 2. Adjuntar una política de permisos

Adjunta una política que permita a Lakehouse leer de Glue y S3.

> [!IMPORTANT] **Alcance seguro de IAM**: el siguiente ejemplo usa estructuras
> con wildcards a modo ilustrativo. **DEBES** consultar con el usuario para
> acotar los ARN de `Resource` a su catálogo, base de datos y buckets de S3
> específicos. NO apliques permisos wildcard a ciegas.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "GlueRead",
      "Effect": "Allow",
      "Action": [
        "glue:GetCatalog",
        "glue:GetDatabase",
        "glue:GetDatabases",
        "glue:GetTable",
        "glue:GetTables"
      ],
      "Resource": "arn:aws:glue:<AWS_REGION>:<AWS_ACCOUNT_ID>:catalog"
    },
    {
      "Sid": "S3Read",
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::<SPECIFIC_BUCKET>",
        "arn:aws:s3:::<SPECIFIC_BUCKET>/*"
      ]
    }
  ]
}
```

Adjunta esta política de permisos a tu rol IAM.

#### 3. Crear el catálogo federado

Al crear un catálogo federado de AWS Glue, el `--glue-warehouse` **DEBE**
configurarse con tu cadena de AWS Account ID de 12 dígitos (no una URI de
bucket de S3). **Buena práctica**: inicializa el catálogo sin especificar una
programación de refresh para evitar fallos prematuros de sincronización de
metadatos mientras las relaciones de confianza de AWS se propagan.

```bash
gcloud alpha biglake iceberg catalogs create <CATALOG_NAME> \
  --project="<PROJECT_ID>" \
  --primary-location="<REGION>" \
  --catalog-type="federated" \
  --federated-catalog-type="glue" \
  --glue-warehouse="<AWS_ACCOUNT_ID>" \
  --glue-aws-region="<AWS_REGION>" \
  --glue-aws-role-arn="arn:aws:iam::<AWS_ACCOUNT_ID>:role/<AWS_ROLE_NAME>"
```

#### 4. Actualizar la trust policy

Extrae el `biglake-service-account-id` del catálogo creado y actualiza la trust
policy de tu rol IAM de AWS para reemplazar `PLACEHOLDER_VALUE` en las
condiciones `aud` y `sub` con este ID de Google Service Agent.

#### 5. Habilitar el refresh en segundo plano

Actualiza el catálogo para activar el refresh en segundo plano una vez
actualizada la trust policy.

```bash
gcloud alpha biglake iceberg catalogs update <CATALOG_NAME> \
  --project="<PROJECT_ID>" \
  --refresh-interval="300s"
```

### Consultar los datos

Una vez configurado, puedes consultar las tablas vía BigQuery.

```sql
SELECT * FROM `<PROJECT_ID>.<CATALOG_NAME>.<NAMESPACE>.<TABLE_NAME>` LIMIT 10;
```

## Gotchas y errores comunes

> [!IMPORTANT] **Aislamiento regional**: el secret de Secret Manager y el
> Lakehouse catalog **DEBEN** crearse en exactamente la misma región.

> [!TIP] **Buena práctica de emparejamiento de regiones**: al configurar el
> catálogo federado, elige regiones de GCP con "Low Latency Dedicated" o
> "Partner CCI" para garantizar un rendimiento óptimo al federar datasets
> grandes entre nubes. Ejemplos de emparejamientos óptimos: - AWS `us-east-1`
> (N. Virginia) se empareja mejor con GCP `us-east4` (Ashburn, VA) - AWS
> `us-west-2` (Oregon) se empareja mejor con GCP `us-west1` (The Dalles, OR) -
> AWS `eu-west-2` (Londres) se empareja mejor con GCP `europe-west2` (Londres) -
> AWS `eu-central-1` (Frankfurt) se empareja mejor con GCP `europe-west3`
> (Frankfurt) Para la lista exhaustiva de mapeos, lee la tabla completa de
> capacidades en:
> https://docs.cloud.google.com/lakehouse/docs/regions-capabilities-cross-cloud-lakehouse

> [!IMPORTANT] **Ubicación de la consulta de BigQuery**: al consultar el
> catálogo federado vía BigQuery, **DEBES** asegurarte de que la consulta se
> ejecute en la misma región que el catálogo (p. ej., `us-east4`). Si usas la
> CLI de `bq`, usa el flag `--location`.

## Paso 3: Validación y siguientes pasos

Después de completar la configuración, el agente **DEBE** validar que la
federación funciona y proponer los siguientes pasos al usuario.

1.  **Valida la conexión**:

    -   Intenta listar los namespaces o las tablas del catálogo recién federado
        usando la CLI de `bq` o la API de BigQuery. Por ejemplo:

        ```bash
        bq ls --location="<REGION>" <PROJECT_ID>.<CATALOG_NAME>
        ```

    -   Si el comando devuelve una lista de namespaces/esquemas, la federación
        fue exitosa.

2.  **Resolución de problemas**:

    -   Si la validación falla (p. ej., errores de permisos, resultados vacíos,
        timeout), el agente debe consultar la documentación de Troubleshooting
        de Cross-Cloud Lakehouse:
        https://docs.cloud.google.com/lakehouse/docs/troubleshooting.
    -   Para AWS Glue, verifica que la trust policy referencie correctamente el
        `biglake-service-account-id` y que las regiones de GCP y AWS coincidan
        con tu configuración.
    -   Para Databricks, verifica que el secret exista en la región correcta y
        que la cuenta de servicio tenga `roles/secretmanager.secretAccessor`.

3.  **Explora y propone**:

    -   Asumiendo que la federación funciona, explora los namespaces
        disponibles y algunas tablas clave.
    -   Resume al usuario qué tipo de datos se encontraron (p. ej., "veo que
        tienes tablas relacionadas con transacciones de e-commerce y perfiles
        de clientes").
    -   Propón al usuario una pregunta de negocio o analítica que dé lugar a una
        consulta significativa de sus datos (p. ej., "¿quieres que escriba una
        consulta para encontrar los 5 clientes que más compraron el mes
        pasado?").
