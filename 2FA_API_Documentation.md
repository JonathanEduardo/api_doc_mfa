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

### 1. Login con Estado 2FA
```http
POST /api/user/auth
```

**Request:**
```json
{
    "employee_number": "52102889",
    "password": "12345678@J",
    "id_device": "test_device",
    "device_os": "android/ios",
    "languageId": 1
}
```

**Response:**
La respuesta incluye toda la información del usuario y además los siguientes campos relacionados con 2FA:
```json
{
    "statuscode": 0,
    "statusmessage": "",
    "resultset": {
        // ... otros campos del usuario antiguos...
        // nuevos valores que agregue
        "has_2fa": false,        // true si el usuario tiene 2FA activado
        "is_first_time": false,  // true si es la primera vez que configura 2FA
        "is_mandatory": true     // true si 2FA es obligatorio para el usuario
    }
}
```

### 2. Generar o Recuperar Secret 2FA
```http
POST /api/generate
```

**Request:**
No requiere cuerpo de petición (n/a)

**Response:**
```json
{
    "statuscode": 0,
    "statusmessage": "OK",
    "resultset": {
        "secret": "BHR2URDQF7FUVYDG",   // Secret en texto plano para configurar en Google Authenticator
        "is_new": false               // true si es un nuevo secret, false si ya existía
    }
}
```

**Notas:**
- Si el usuario ya tiene un secret configurado, se retornará el mismo (is_new: false)
- Si el usuario no tiene secret, se generará uno nuevo (is_new: true)
- El secret debe ingresarse manualmente en Google Authenticator

### 3. Validar Código
```http
POST /api/validate
```

**Request:**
```json
{
    "code": "883036"     // Código 2FA de 6 dígitos
}
```

**Response:**
```json
{
    "statuscode": 0,
    "statusmessage": "OK",
    "resultset": {
        "status": "success"  // Cuando el código es válido
    }
}
```

### 4. Desactivar 2FA
```http
POST /api/disable
```

**Request:**
```json
{
    "code": "034166"     // Código 2FA de 6 dígitos para confirmar
}
```

**Response:**
```json
{
    "statuscode": 0,
    "statusmessage": "OK",
    "resultset": {
        "status": "success"  // Cuando se desactivó correctamente
    }
}
```

## Códigos de Error

Todos los endpoints pueden devolver los siguientes errores:

```json
// Error de autenticación - Usuario inválido
{
    "statuscode": 4,
    "statusmessage": "Invalid user",
    "resultset": []
}

// Error de servidor
{
    "statuscode": 2,
    "statusmessage": "Server error",
    "resultset": []
}
```

### Errores Específicos por Endpoint

#### Generar Secret (/api/generate)
```json
// Error de servidor al generar secret
{
    "statuscode": 2,
    "statusmessage": "Server error",
    "resultset": []
}
```

#### Validar Código (/api/validate)
```json
// Error - 2FA no configurado
{
    "statuscode": 1,
    "statusmessage": "2FA is not configured",
    "resultset": []
}

// Error - Código inválido
{
    "statuscode": 1,
    "statusmessage": "Invalid verification code",
    "resultset": []
}

// Error de servidor
{
    "statuscode": 2,
    "statusmessage": "Server error",
    "resultset": []
}

// Error de validación
{
    "statuscode": 1,
    "statusmessage": "The code field is required",
    "resultset": []
}
```

#### Desactivar 2FA (/api/disable)
```json
// Error - 2FA no configurado
{
    "statuscode": 1,
    "statusmessage": "2FA is not configured",
    "resultset": []
}

// Error - Código inválido
{
    "statuscode": 1,
    "statusmessage": "Invalid verification code",
    "resultset": []
}

// Error de servidor
{
    "statuscode": 2,
    "statusmessage": "Server error",
    "resultset": []
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
