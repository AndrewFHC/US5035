# Historia de Usuario: Inicio de sesión en Galaxy One

## Historia

**Como** usuario (Asesor de negocio, Asesor líder, Administrador de tienda, Jefe zonal)  
**Quiero** iniciar sesión en el sistema Galaxy One usando mis credenciales de Microsoft Entra ID con autenticación multifactor  
**Para** acceder a mis funcionalidades, menú y dashboard según mi rol

---

## Criterios de aceptación (formato Gherkin)

```gherkin
Escenario: Inicio de sesión exitoso
  Dado que el usuario está en la pantalla de bienvenida
  Cuando selecciona “Iniciar sesión con Microsoft”
  Y proporciona un correo válido registrado en BD
  Y proporciona la contraseña correcta
  Y recibe y valida el código MFA
  Entonces el sistema le permite el acceso a Galaxy One

Escenario: Correo inválido
  Dado que el usuario ingresa un correo con formato incorrecto o no registrado
  Entonces el sistema muestra un mensaje de error

Escenario: Contraseña incorrecta
  Dado que el usuario ingresa una contraseña inválida
  Entonces el sistema muestra un mensaje de error

Escenario: Código MFA incorrecto o expirado
  Dado que el usuario ingresa un código inválido o expirado
  Entonces el sistema muestra un mensaje de error informativo

Escenario: Reenvío de código
  Dado que el usuario no recibe el código
  Entonces puede solicitar el reenvío al correo electrónico

Escenario: Rechazo de acceso
  Dado que el usuario ingresa el código correcto pero selecciona “No, no soy yo”
  Entonces el sistema bloquea el acceso

Escenario: Navegación hacia atrás
  Dado que el usuario está en la pantalla de ingreso de correo
  Cuando selecciona “Atrás”
  Entonces el sistema lo redirige a la pantalla de bienvenida

Escenario: Registro de intentos
  Dado cualquier intento de inicio de sesión
  Entonces el sistema registra fecha, IP, resultado, error (si aplica) y georreferencia
```

## Reglas de negocio

- El correo debe tener formato válido.
- La autenticación se realiza mediante Microsoft Entra ID con MFA.
- Todos los intentos de acceso (exitosos o fallidos) deben registrarse con:
  - Fecha y hora
  - IP del dispositivo
  - Resultado (Exitoso/Fallido)
  - Detalle del error (si aplica)
  - Georreferencia (latitud/longitud)
- Si el usuario rechaza el acceso (“No, no soy yo”), se bloquea la sesión.
- El sistema debe permitir reenviar el código MFA si no se recibe.

---

## Escenarios posibles

- Inicio de sesión exitoso
- Correo inválido
- Contraseña incorrecta
- Código MFA incorrecto o expirado
- Rechazo de acceso
- Reenvío de código
- Navegación hacia atrás
- Demora en carga de datos (>10 seg)
- Registro de intentos


## Request / Response

Request: Inicio de sesión
```
POST /auth/login
{
  "email": "usuario@empresa.com"
}

```

Response: Inicio de sesión
```
{
  "status": "next_step",
  "step": "password"
}
```

Request: Validación MFA
```
POST /auth/verify
{
  "email": "usuario@empresa.com",
  "password": "********",
  "mfa_code": "123456"
}

```

Response: Validación MFA
```
{
  "status": "success",
  "token": "jwt-token",
  "role": "Administrador de tienda"
}

```

## Tabla de Mensajes del Sistema: Autenticación Galaxy One

| Tipo de mensaje | Condición del sistema                                      | Mensaje mostrado al usuario                                      |
|-----------------|------------------------------------------------------------|------------------------------------------------------------------|
| ✅ Éxito         | Inicio de sesión completado correctamente                  | "Autenticación exitosa. Bienvenido a Galaxy One."                |
| ✅ Éxito         | Código MFA validado y usuario confirma identidad           | "Código verificado. Accediendo al sistema..."                    |
| ✅ Éxito         | Código reenviado correctamente                             | "Se ha reenviado el código de verificación a tu correo."         |
| ⚠️ Error         | Correo electrónico con formato inválido                    | "El correo ingresado no tiene un formato válido."                |
| ⚠️ Error         | Correo no registrado en la base de datos                   | "El correo ingresado no está registrado."                        |
| ⚠️ Error         | Contraseña incorrecta                                      | "La contraseña ingresada es incorrecta."                         |
| ⚠️ Error         | Código MFA incorrecto                                      | "El código ingresado no es válido. Intenta nuevamente."          |
| ⚠️ Error         | Código MFA expirado                                        | "El código ha expirado. Solicita uno nuevo."                     |
| ⚠️ Error         | Usuario selecciona “No, no soy yo”                         | "Acceso denegado. Verifica tu identidad."                        |
| ⚠️ Error         | Usuario no ingresa el código MFA                           | "Debes ingresar el código de verificación para continuar."       |
| ⚠️ Error         | Fallo en la verificación de credenciales                   | "No se pudo verificar tu identidad. Revisa tus datos."           |
| ⚠️ Error         | Demora excesiva en carga de datos (>10 segundos)          | "Estamos tardando más de lo esperado. Por favor, espera..."      |
| ⚠️ Error         | Fallo inesperado del sistema                               | "Ha ocurrido un error inesperado. Intenta nuevamente más tarde." |

