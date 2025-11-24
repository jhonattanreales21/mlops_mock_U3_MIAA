# Changelog

Todos los cambios notables en este proyecto serán documentados en este archivo.

El formato se basa en [Keep a Changelog](https://keepachangelog.com/es-ES/1.0.0/),

## [1.0.0] - 2025-11-22
**Versión Final de la Propuesta (AWS MLOps Enterprise Architecture)**

Se define la arquitectura final orientada a servicios gestionados en la nube (AWS) con soporte para despliegue híbrido, seguridad HIPAA/GDPR y manejo multimodal de datos.

### Evolución de la Arquitectura
Comparativa entre la visión inicial conceptual y la solución final implementada:

| Aspecto | Versión Inicial (Semana 1) | Versión Final (AWS MLOps) |
| :--- | :--- | :--- |
| **Infraestructura** | General / Conceptual | **AWS:** S3, EMR, ECS, SageMaker, ECR, CloudWatch |
| **Procesamiento** | Descripción amplia | **PySpark + EMR + Airflow (MWAA)** |
| **Entrenamiento** | Python general | **SageMaker Pipelines** + Python/PySpark |
| **Versionamiento** | Ideas generales | **SageMaker Model Registry** & Feature Store |
| **CI/CD** | Mención general | **GitHub Actions** + ECR + ECS/Fargate |
| **Despliegue** | API conceptual | **Híbrido:** Cloud (FastAPI en ECS) y Local (ONNX + Docker) |
| **Monitoreo** | General | **CloudWatch + SageMaker Model Monitor** |
| **Seguridad** | General | **IAM + KMS + VPC + CloudTrail** |

### Added (Añadido)
- **Despliegue Híbrido:** Estrategia dual para operar en la nube (ECS) y en local (contenedores con ONNX Runtime) para hospitales sin conexión.
- **Validación de Datos Robusta:** Implementación de *Great Expectations* sobre *SageMaker Processing Jobs*.
- **Orquestación Completa:** Definición de DAGs en Airflow (MWAA) para coordinar ingesta, reentrenamiento y monitoreo.
- **Componentes de Seguridad:** Integración explícita de cifrado KMS, roles IAM y auditoría con CloudTrail para cumplimiento normativo.

## [0.2.0] - 2025-11-15
**MVP Técnico (Prueba de Concepto)**

Implementación técnica de un Producto Mínimo Viable (MVP) para validar flujos de CI/CD y consumo de modelos.
*Repositorio de referencia:* [`ml-med-app-mlops-U2`](https://github.com/jhonattanreales21/ml-med-app-mlops-U2)

### Added
- **Web App:** Desarrollo de interfaz de usuario utilizando **Streamlit** para interacción con el modelo.
- **CI/CD Pipeline:** Configuración de workflows en **GitHub Actions** para automatizar pruebas y construcción.
- **Prototipado:** Implementación de código funcional para demostrar la viabilidad de la inferencia.

## [0.1.0] - 2025-10-20
**Propuesta Inicial (Diseño Conceptual)**

Definición teórica del problema y esquema general del pipeline sin vinculación a un proveedor de nube específico.

### Added
- **Definición del Problema:** Diagnóstico de enfermedades clasificadas por severidad (No enfermo, Leve, Aguda, Crónica).
- **Alcance de Enfermedades:** Distinción entre enfermedades comunes (Big Data) y enfermedades huérfanas (Small Data).
- **Tipos de Datos:** Identificación de fuentes tabulares, texto clínico e imágenes médicas.
- **Estrategias de Modelado (Baseline):**
    - Tabular: Regresión Logística, XGBoost.
    - Texto: ClinicalBERT.
    - Imágenes: Transfer Learning.
- **Requerimientos No Funcionales:** Definición inicial de privacidad, latencia y explicabilidad.
- **Esquema General:** Diagrama de bloques abarcando Diseño, Desarrollo, Producción y Gobierno.