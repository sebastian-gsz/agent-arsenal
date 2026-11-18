---
name: google-secure-coder
description: Hub central de la suite de Google para datos en GCP y seguridad de código. Úsala como punto de entrada cuando el trabajo toque Google Cloud (BigQuery, Spark, Dataflow, Cloud Composer, catálogos de datos), seguridad de código, o cuando no sepas qué skill específica aplicar — el hub enruta al módulo y skill correctos.
---

# Google Secure Coder — Hub central

Suite dividida en módulos temáticos bajo `modules/`. Cada módulo agrupa skills orientadas a una necesidad concreta. **No intentes resolver la tarea aquí:** identifica el módulo, lee su SKILL.md o índice, y delega a la skill específica.

## Mapa de módulos

| Módulo | Necesidad que cubre | Skills |
|---|---|---|
| `modules/bigquery/` | Analizar, transformar y servir datos en BigQuery: BQML, vector search, optimización de queries, pipelines dbt y Dataform, transfer service, apps de datos | developing-with-bigquery, dbt-bigquery, dataform-bigquery, bigquery-data-transfer-service, building-data-apps |
| `modules/pipelines/` | Construir y operar pipelines de datos en GCP: entry point general, Spark en Dataproc, Dataflow, orquestación, aprovisionamiento de recursos, troubleshooting de Composer | gcp-data-pipelines (entry point), gcp-spark, gcp-dataflow, gcp-pipeline-orchestration, gcp-pipeline-resource-provisioning, gcp-composer-troubleshooting |
| `modules/catalogo-calidad/` | Descubrir y gobernar datos: encontrar assets, catálogos federados lakehouse, calidad de datos automatizada | discovering-gcp-data-assets, federate-lakehouse-catalog, data-autocleaning |
| `modules/securecoder/` | Seguridad de código: modelado de amenazas, escaneo, informe de auditoría, remediación. **Ojo:** estas skills asumen el producto SecureCoder (API local); fuera de él úsalas como metodología, no como flujo ejecutable | securecoder-persona, determine-threat-model, run-security-scanner, scan-dependencies, generate-security-audit-report, create-security-implementation-plan, run-poc, mandatory-secure-web-skills |
| `modules/soporte/` | Higiene transversal: verificación de credenciales gcloud, gestión de dependencias Python, ML best practices, notebooks, reparación de skills | gcloud-auth-verification, managing-python-dependencies, ml-best-practices, notebook-guidance, skill-repair |

## Cómo enrutar

1. **Pipelines en GCP:** empieza siempre por `modules/pipelines/gcp-data-pipelines/SKILL.md` — es el entry point oficial y deriva a Spark, Dataflow u orquestación según el caso.
2. **Datos en BigQuery:** según la tarea — explorar (`developing-with-bigquery`), transformaciones versionadas (`dbt-bigquery` o `dataform-bigquery`), mover datos entre sistemas (`bigquery-data-transfer-service`), exponer en una app (`building-data-apps`).
3. **Antes de operar sobre GCP:** si hay dudas de autenticación, pasa primero por `modules/soporte/gcloud-auth-verification`.
4. **Calidad de datos:** para scans de perfilado usa el script incluido en `modules/catalogo-calidad/data-autocleaning/scripts/dataplex_scanner.py`.
5. **Seguridad:** sin el producto SecureCoder instalado, usa las skills del módulo como checklist metodológica (modelo de amenazas → scan → informe → remediación) adaptando el scanner a herramientas CLI disponibles (ej. Semgrep).

## Regla general de dependencias

Varias skills de este hub asumen herramientas de Google Cloud (gcloud, bq, bibliotecas de cliente). Antes de ejecutar comandos que muten recursos de GCP, verifica autenticación y proyecto activo — nunca asumas credenciales de sesiones previas.
