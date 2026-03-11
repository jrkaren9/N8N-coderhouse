# Workflow: Consulta de definiciones y traducciones con envío por email

## Nombre del caso elegido
Obtención de definiciones y traducciones desde Wiktionary.

## Resultado esperado
Si el payload es válido, el workflow consulta la API de diccionario para cada palabra, procesa definiciones y traducciones, genera un cuerpo de email en HTML y envía un único correo al destinatario indicado.

## Qué dispara el workflow (trigger)
El workflow se dispara a través de un **Webhook** que recibe una solicitud HTTP `POST` con un payload JSON.

### Estructura esperada del payload

```json
{
  "body": {
    "name": "Karen",
    "email": "karen.justo@hotmail.com",
    "words": ["hello", "incredible"]
  }
}
```
## Descripción de cada nodo

### 1. Webhook
Recibe la solicitud externa y actúa como punto de entrada del workflow.  

### 2. Handle input
Evalúa si el payload recibido contiene la información mínima necesaria para continuar.  

### 3. Obtain data
Normaliza y conserva los datos originales recibidos desde el webhook.  

### 4. Split Out
Toma el arreglo de palabras y lo separa en items individuales.  

### 5. Wiktionary (HTTP Request)
Realiza una llamada a la API externa para obtener definiciones, categoría gramatical y traducciones de cada palabra.  

Cada item generado por `Split Out` produce una consulta independiente.

### 6. Process results
Procesa la respuesta de la API para cada palabra.   Extrae definiciones, parte de la oración y traducciones al coreano y al español, y además genera un bloque HTML legible por palabra.

### 7. Generate email body
Une los bloques procesados de todas las palabras en un único cuerpo de email.  

### 8. Gmail / Send a message
Envía un correo electrónico con el resumen de definiciones y traducciones.  

### 9. Respond to Webhook
Devuelve una respuesta HTTP al cliente que llamó al webhook.  

Se usa para informar si la ejecución fue exitosa o si hubo un error de validación.

## Qué evalúa el condicional y por qué
El condicional verifica si el webhook recibió los campos mínimos requeridos para procesar la solicitud correctamente:

- `body.name`
- `body.email`
- `body.words`

La validación existe para evitar ejecutar llamadas innecesarias a la API o intentar enviar un correo cuando faltan datos obligatorios.

## Cómo configuré la notificación
La notificación se configuró con el nodo de **Gmail** en modo de envío de mensaje.  

El destinatario se toma del email recibido en el webhook, el asunto se personaliza con el nombre del usuario y el cuerpo del mensaje se genera en formato **HTML** para que la información quede más legible. 

La autenticación se hace por medio de OAuth.

## Buenas prácticas aplicadas

### Credenciales
Las credenciales de Gmail se manejan mediante el sistema de credenciales de n8n y no se escriben directamente dentro del flujo.

### Logs
Durante el desarrollo se utilizaron `console.log()` en los nodos de código para validar la estructura real de los items y detectar problemas de conexiones entre ramas o rutas incorrectas.

### Idempotencia
El flujo procesa la información recibida sin modificar registros externos, salvo el envío del correo. 

Como mejora futura, podría agregarse un control para evitar correos duplicados si el mismo payload se recibe más de una vez.

## Manejo de errores / datos faltantes
Se intentó contemplar el caso en que faltaran datos requeridos en el payload. 

La idea fue validar tempranamente el contenido recibido y responder con un error claro si no estaban presentes `name`, `email` o `words`.

Aunque esta validación quedó parcialmente implementada, la mejora pendiente sería consolidar todas las ramas inválidas para devolver respuestas HTTP controladas, por ejemplo:

- `400 Bad Request` si falta el nombre
- `400 Bad Request` si falta el email
- `400 Bad Request` si no se recibió el arreglo de palabras