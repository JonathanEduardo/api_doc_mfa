# Documentación API 2FA - KIA Contigo

## Headers Requeridos
Todos los endpoints requieren los siguientes headers:

```http
Authorization: {token}
Content-Type: application/json
Accept: application/json
```

Nota: El token debe enviarse directamente en el header Authorization, sin el prefijo "Bearer".

## Endpoints

### 1. Obtener Estado 2FA
```http
GET /api/status
```

**Response:**
```json
{
    "has_2fa": false,        // true si 2FA está activado
    "is_first_time": true,   // true si es primera vez activando 2FA
    "is_mandatory": true     // true si 2FA es obligatorio
}
```

### 2. Generar Nuevo Secreto
```http
POST /api/generate
```

**Response:**
```json
{
    "url": "otpauth://totp/KIA%20Contigo:52102889?secret=LC5RCZ3SIX6NRKQ7&issuer=KIA%20Contigo&algorithm=SHA1&digits=6&period=30"
}
```

### 3. Obtener Código QR
```http
GET /api/qr-code
```

**Response:**
```json
{
    "qr_url": "otpauth://totp/KIA%20App:52102889?secret=LC5RCZ3SIX6NRKQ7&issuer=KIA%20App&algorithm=SHA1&digits=6&period=30"
}
```

### 4. Validar Código
```http
POST /api/validate
```

**Request (raw):**
```json
{
    "code": "632570"     // Código 2FA de 6 dígitos
}
```

**Response:**
```json
{
    "status": "success"  // Cuando el código es válido
}
```

### 5. Desactivar 2FA
```http
POST /api/disable
```

**Request (raw):**
```json
{
    "code": "034166"     // Código 2FA de 6 dígitos para confirmar
}
```

**Response:**
```json
{
    "status": "success"  // Cuando se desactivó correctamente
}
```

## Códigos de Error

Todos los endpoints pueden devolver los siguientes errores:

```json
// Error 401 - Token inválido
{
    "statuscode": 4,
    "statusmessage": "Invalid user",
    "resultset": []
}

// Error 404 - Usuario no encontrado
{
    "error": "User not found"
}
```

### Errores Específicos por Endpoint

#### Validar Código (/api/validate)
```json
// Error 400 - 2FA no configurado
{
    "error": "2FA is not configured"
}

// Error 422 - Código inválido
{
    "error": "Invalid code"
}

// Error 500 - Error interno
{
    "error": "Error validating code"
}

// Error de validación
{
    "message": "The code field is required.",
    "errors": {
        "code": [
            "The code field is required."
        ]
    }
}
```

#### Desactivar 2FA (/api/disable)
```json
// Error 400 - 2FA no configurado
{
    "error": "2FA is not configured"
}

// Error 422 - Código inválido
{
    "error": "Invalid code"
}

// Error 500 - Error interno
{
    "error": "Error validating code"
}
```

#### Obtener Código QR (/api/qr-code)
```json
// Error 400 - 2FA no configurado
{
    "error": "2FA not enabled for this user"
}

// Error 500 - Error interno
{
    "error": "Error processing 2FA secret"
}
```

## Notas de Seguridad

1. El secreto 2FA se almacena encriptado en la base de datos
2. Nunca se expone el secreto en las respuestas de la API
3. La validación de códigos se realiza con el secreto desencriptado de forma segura
4. Los códigos 2FA son siempre de 6 dígitos
5. Se utiliza el algoritmo SHA1 para la generación de códigos
6. El período de validez de cada código es de 30 segundos

## Respuestas a Preguntas del Desarrollador

### 1. Cambio de Contraseña con 2FA
El flujo para cambio de contraseña cuando 2FA está activo es el siguiente:

1. Usuario solicita cambio de contraseña
2. Se valida primero la contraseña actual
3. Se solicita código 2FA para confirmar la identidad
4. Una vez validado el código 2FA, se permite el cambio de contraseña
5. El 2FA permanece activo después del cambio de contraseña

### 2. Desactivación de 2FA
Existe un endpoint específico para desactivar 2FA (`POST /api/disable`). Este servicio:
- Requiere un código 2FA válido para confirmar la desactivación
- Elimina el secreto 2FA del usuario
- Resetea el estado de primera vez
- Devuelve confirmación de desactivación
