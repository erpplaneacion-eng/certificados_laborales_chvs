# Proyecto: Aplicación de Generación de Certificados Laborales

Este proyecto es una aplicación web sofisticada construida con **FastAPI** diseñada para automatizar la generación de certificados laborales profesionales en formato PDF, integrándose con el ecosistema de Google (Sheets y Drive).

## Arquitectura y Tecnologías

### Core Stack
- **Framework Web:** FastAPI (Python 3.11+)
- **Generación de PDF:** ReportLab (Platypus)
- **Base de Datos:** Google Sheets (vía `gspread`)
- **Almacenamiento:** Google Drive (vía `google-api-python-client`)
- **Conversión de Números:** `num2words` (Configurado para español)
- **Configuración:** `pydantic-settings` para gestión de variables de entorno

### Estructura del Proyecto
- `app/main.py`: Punto de entrada, rutas y lógica de orquestación.
- `app/config.py`: Definición de configuraciones y carga de `.env`.
- `app/google_clients.py`: Fábrica de clientes autenticados para Google APIs.
- `app/services/`:
  - `sheets_service.py`: Lógica para consulta de registros y normalización de empresas.
  - `drive_service.py`: Lógica para subida de archivos PDF a carpetas específicas.
  - `template.py`: Diseño y maquetación de los certificados PDF (márgenes, firmas, estilos).
- `app/templates/`: Plantillas HTML (Jinja2) para la interfaz de usuario.
- `app/static/`: Recursos estáticos como logotipos (SVG).
- `firma/`: Contiene la imagen de la firma digitalizada (`firma.png`) utilizada en los PDFs.

## Flujos de Trabajo Principales

### 1. Verificación de Cédula
El usuario ingresa una cédula; la aplicación consulta la hoja `bd_contratacion` en Google Sheets para verificar si el empleado existe, retornando su último cargo y estado del contrato.

### 2. Generación de Certificados
- Se consultan todos los registros históricos del empleado.
- Se normalizan los nombres de las empresas usando la hoja `Empresas` (mapeo de alias a nombres canónicos y NITs).
- Se agrupan los contratos por empresa canónica.
- Se genera un PDF consolidado por cada empresa, aplicando lógica condicional (salarios para cargos específicos, textos para programas como PAE, etc.).
- Los PDFs se suben a una carpeta de Google Drive configurada.

## Configuración del Entorno

### Variables de Entorno (.env)
Se requiere un archivo `.env` con las siguientes claves:
- `GOOGLE_CREDENTIALS_JSON`: Credenciales de la cuenta de servicio de Google en formato JSON.
- `SHEET_ID`: ID de la hoja de cálculo de Google Sheets.
- `DRIVE_FOLDER_ID`: ID de la carpeta de destino en Google Drive.
- `PORT`: Puerto para ejecutar la aplicación (por defecto 8000).

### Comandos de Desarrollo
- **Ejecutar localmente:** `uvicorn app.main:app --reload`
- **Instalar dependencias:** `pip install -r requirements.txt`

## Despliegue e Infraestructura

### Docker
La aplicación incluye un `Dockerfile` optimizado (basado en `python:3.11-slim`) que:
- Configura los locales para soporte completo de español (`es_ES.UTF-8`).
- Instala dependencias del sistema y de Python.
- Copia el código y los recursos de firma.
- Expone el puerto 10000 (estándar para servicios como Render).

### Render / Cloud
- `Procfile`: Define el comando de inicio para la plataforma.
- `build.sh`: Script de construcción para configurar el entorno de ejecución (locales y dependencias).

## Convenciones de Desarrollo
- **Idiomas:** El código utiliza nombres de funciones y variables en español e inglés (mezclado, pero coherente en servicios). Los mensajes al usuario y los documentos generados son estrictamente en español.
- **Normalización:** Siempre consultar `sheets_service.get_company_info_lookup()` antes de procesar nombres de empresas para asegurar que se usen los nombres oficiales y NITs correctos.
- **Firmas:** La lógica de firmas en `app/services/template.py` busca la imagen en la raíz del proyecto (`/firma/firma.png`).
