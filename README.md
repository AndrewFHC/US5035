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

## Escenario 1: Inicio de sesión exitoso

| Elemento        | Detalle                                                                 |
|-----------------|-------------------------------------------------------------------------|
| Condición       | Usuario ingresa correo válido, contraseña correcta y código MFA válido |
| Resultado       | Acceso permitido                                                        |
| Mensaje         | "Autenticación exitosa. Bienvenido a Galaxy One."                       |

---

## Escenario 2: Correo inválido

| Elemento        | Detalle                                                                 |
|-----------------|-------------------------------------------------------------------------|
| Condición       | Usuario ingresa correo con formato incorrecto o no registrado           |
| Resultado       | Acceso denegado                                                         |
| Mensaje         | "El correo ingresado no tiene un formato válido o no está registrado."  |

---

## Escenario 3: Contraseña incorrecta

| Elemento        | Detalle                                                                 |
|-----------------|-------------------------------------------------------------------------|
| Condición       | Usuario ingresa contraseña inválida                                     |
| Resultado       | Acceso denegado                                                         |
| Mensaje         | "La contraseña ingresada es incorrecta."                                |

---

## Escenario 4: Código MFA incorrecto

| Elemento        | Detalle                                                                 |
|-----------------|-------------------------------------------------------------------------|
| Condición       | Usuario ingresa código MFA inválido                                     |
| Resultado       | Acceso denegado                                                         |
| Mensaje         | "El código ingresado no es válido. Intenta nuevamente."                 |

---

## Escenario 5: Código MFA expirado

| Elemento        | Detalle                                                                 |
|-----------------|-------------------------------------------------------------------------|
| Condición       | Código MFA ingresado ha expirado                                        |
| Resultado       | Acceso denegado                                                         |
| Mensaje         | "El código ha expirado. Solicita uno nuevo."                            |

---

## Escenario 6: Rechazo de acceso

| Elemento        | Detalle                                                                 |
|-----------------|-------------------------------------------------------------------------|
| Condición       | Usuario ingresa código correcto pero selecciona “No, no soy yo”         |
| Resultado       | Acceso bloqueado                                                        |
| Mensaje         | "Acceso denegado. Verifica tu identidad."                               |

---

## Escenario 7: Reenvío de código

| Elemento        | Detalle                                                                 |
|-----------------|-------------------------------------------------------------------------|
| Condición       | Usuario no recibe el código y solicita reenviarlo                       |
| Resultado       | Código reenviado                                                        |
| Mensaje         | "Se ha reenviado el código de verificación a tu correo."                |

---

## Escenario 8: Navegación hacia atrás

| Elemento        | Detalle                                                                 |
|-----------------|-------------------------------------------------------------------------|
| Condición       | Usuario selecciona “Atrás” desde pantalla de ingreso de correo          |
| Resultado       | Redirección a pantalla de bienvenida                                    |
| Mensaje         | "Regresando a la pantalla de bienvenida."                               |

---

## Escenario 9: Demora en carga de datos

| Elemento        | Detalle                                                                 |
|-----------------|-------------------------------------------------------------------------|
| Condición       | El sistema tarda más de 10 segundos en responder                        |
| Resultado       | Mensaje informativo                                                     |
| Mensaje         | "Estamos tardando más de lo esperado. Por favor, espera..."             |

---

## Escenario 10: Registro de intentos

| Elemento        | Detalle                                                                 |
|-----------------|-------------------------------------------------------------------------|
| Condición       | Cualquier intento de inicio de sesión                                   |
| Resultado       | Registro en BD con metadatos                                            |
| Datos registrados | Fecha, hora, IP, resultado, error (si aplica), georreferencia         |



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

