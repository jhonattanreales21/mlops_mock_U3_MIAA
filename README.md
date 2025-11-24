# MLOps Pipeline: Diagnóstico de Enfermedades Asistido por Machine Learning

![Status](https://img.shields.io/badge/Status-Proposal-blue) ![Domain](https://img.shields.io/badge/Domain-Healthcare-red)

## 1. Desafio Pipeline MLOps (sector salud)

### Contexto
En el contexto médico actual, la información de los pacientes ha crecido exponencialmente (historias clínicas, imágenes, notas), formando un ecosistema rico pero heterogéneo. Sin embargo, para patologías poco frecuentes (enfermedades huérfanas), la disponibilidad de datos es limitada, dificultando la creación de modelos robustos.

### Definición del Problema
El objetivo es desarrollar y operar un sistema inteligente de soporte a la decisión clínica capaz de predecir la probabilidad de que un paciente padezca una enfermedad y su nivel de severidad.

* **Usuario Final:** Médicos especialistas y hospitales.
* **Tarea de Predicción:** Clasificar al paciente en 5 niveles de severidad: *No Enfermo, Leve, Aguda, Crónica, Terminal*.
* **Reto Principal:** Manejar tanto enfermedades comunes (muchos datos) como huérfanas (pocos datos) bajo estrictas normas de privacidad y seguridad.

---

## 2. Diseño del Pipeline de Machine Learning

Para abordar este problema, se propone una arquitectura **End-to-End en AWS**, diseñada para soportar el ciclo de vida completo del modelo, desde la ingesta de datos heterogéneos hasta un despliegue híbrido (Nube y Local). Para información mas detallada del trabajo desarrollado, consultar el siguiente [archivo.][./archivos/informe_propuesta_mlops]

### Visión General de la Arquitectura
La solución se divide en dos secciones principales: **Ingesta/Entrenamiento** y **Despliegue/Inferencia**.

#### Sección 1: Datos, Calidad y Entrenamiento (Offline)
El siguiente diagrama detalla el flujo de datos, desde las fuentes clínicas (EHR, PACS) hasta el registro del modelo, pasando por validación de calidad y *feature engineering*.

![Data & Training Pipeline](./archivos/Diagram_1st_section_ghub.jpg)

**Componentes Clave:**

1.  **Ingesta y Data Lake (S3):**
    * Almacenamiento seguro de datos crudos, procesados y etiquetados en Amazon S3.
    * Soporte para datos tabulares (HL7/FHIR), imágenes (DICOM) y notas médicas.

2.  **Calidad y Etiquetado:**
    * **Validación:** Uso de *Great Expectations* y *SageMaker Processing Jobs* para asegurar la calidad de los datos y rangos fisiológicos.
    * **Ground Truth:** *SageMaker Ground Truth* gestiona el etiquetado clínico con trazabilidad y *Human-in-the-loop*.

3.  **Feature Store:**
    * Centralización de variables (features) en *SageMaker Feature Store* para garantizar consistencia entre entrenamiento y producción.

4.  **Pipeline de Entrenamiento Multimodal:**
    * Estrategia de entrenamiento que combina modelos para datos tabulares (XGBoost), texto (Clinical BERT) e imágenes (CNN).
    * Manejo de desbalance de clases para enfermedades huérfanas mediante técnicas de *oversampling* y pesos de clases.
    * Orquestación mediante *SageMaker Pipelines* y *Airflow*.

---

#### Sección 2: Despliegue e Inferencia (Online/Híbrido)
El sistema debe funcionar tanto en hospitales con alta conectividad como en consultorios locales con restricciones de red.

![Deployment & Inference Pipeline](./archivos/Diagram_2nd_section_ghub.jpg)

**Estrategias de Despliegue:**

* **Cloud Deployment (Alta Disponibilidad):**
    * Microservicio basado en **FastAPI** desplegado en **Amazon ECS/Fargate**.
    * Expuesto vía **API Gateway** con seguridad HTTPS/TLS y autenticación IAM.
    * Ideal para hospitales grandes e integraciones centralizadas.

* **Local / On-Premise Deployment (Baja Latencia/Privacidad):**
    * Exportación del modelo a formato **ONNX** para portabilidad.
    * Empaquetado en un contenedor Docker ligero para ejecución en servidores locales o laptops médicos sin dependencia de internet.

---

## 3. Implementación y Tecnologías

El stack tecnológico ha sido seleccionado minuciosamente para cumplir con los requisitos de **HIPAA/GDPR**, escalabilidad y operación híbrida.

| Servicio / Herramienta | Rol en el pipeline | Justificación |
| :--- | :--- | :--- |
| **Amazon S3** | Data Lake & Artefactos | Almacenamiento durable, económico y estándar para todo el ciclo de vida del dato. |
| **SageMaker Processing + Great Expectations** | Validación y Calidad | Permite jobs reproducibles y aplicación de reglas de calidad formales sobre datos sensibles. |
| **SageMaker Ground Truth** | Etiquetado Clínico | Facilita el etiquetado supervisado por médicos con trazabilidad y control de calidad. |
| **SageMaker Feature Store** | Gestión de Features | Garantiza consistencia de variables entre entrenamiento y producción (evita *training-serving skew*). |
| **SageMaker Pipelines** | Orquestación ML | Pipelines declarativos, reproducibles y auditables nativos para AWS. |
| **SageMaker Model Registry** | Gobierno de Modelos | Asegura trazabilidad de versiones, métricas y facilita la promoción a producción. |
| **AWS ECR + ECS/Fargate** | Contenedores Cloud | Ejecución *serverless* del microservicio de inferencia con escalabilidad automática. |
| **API Gateway + HTTPS/TLS** | Exposición Segura | Control de acceso, *throttling* y cifrado en tránsito para proteger la API clínica. |
| **Frontend React + S3/CloudFront** | Interfaz de Usuario | Entrega rápida y segura de la UI para los médicos con baja latencia. |
| **ONNX Runtime + Docker Local** | Inferencia On-Premise | Permite ejecutar el modelo en hospitales sin internet, manteniendo portabilidad. |
| **MWAA (Airflow)** | Orquestación General | Coordinación de flujos batch (ETL, monitoreo) mediante DAGs reprogramables. |
| **CloudWatch + Model Monitor** | Monitoreo | Visibilidad centralizada de logs, latencia y detección de *drift* en producción. |
| **AWS CloudTrail** | Auditoría | Soporte regulatorio (HIPAA) con trazabilidad de quién hizo qué y cuándo. |
| **IAM + VPC + KMS** | Seguridad Transversal | Obligatorio para datos clínicos: *least privilege*, aislamiento de red y cifrado *end-to-end*. |
| **GitHub + GitHub Actions** | CI/CD | Integración continua, pruebas automáticas y despliegue controlado a AWS. |

### Monitoreo y Reentrenamiento (Drift Detection)
El sistema incluye un módulo de monitoreo continuo:
* Detección de **Data Drift** (cambios en la población de pacientes) y **Model Drift** (degradación del desempeño) usando *SageMaker Model Monitor*.
* Disparadores automáticos para reentrenamiento cuando las métricas clínicas (Sensibilidad/Especificidad) caen por debajo del umbral.

---

## 4. Cómo ejecutar la propuesta (Simulación)

Contamos con un [segundo repositorio][https://github.com/jhonattanreales21/ml-med-app-mlops-U2] en el cual se encuentra un trabajo en progreso de lo aqui planteado. Vale la pena aclarar que es una prueba de concepto minima. Este repositorio cuenta con una serie de pasos para ejecutar la aplicación via contenedores de docker.

---

## 5. Escenarios Futuros y Escalabilidad

La arquitectura está diseñada para evolucionar según nuevos requerimientos:

1.  **Predicciones en Tiempo Real (<3s):** La infraestructura en ECS/Fargate permite auto-escalado horizontal para manejar picos de demanda en hospitales grandes.
2.  **Feedback del Usuario (Médico):** Se implementará un *loop* de retroalimentación donde el médico pueda confirmar o corregir el diagnóstico sugerido, datos que se usarán para el reentrenamiento futuro.
3.  **Federated Learning:** Para mejorar la privacidad en enfermedades raras, se podría explorar entrenamiento federado sin mover los datos de los hospitales.

---

### Autores
* **Andrés Felipe Cano Larrahondo**
* **Jhonattan Rafael Reales De La Asunción**
* *Universidad ICESI - Maestría en Inteligencia Artificial Aplicada*