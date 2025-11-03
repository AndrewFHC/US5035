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

## Diagrama de Secuencia: Autenticación Galaxy One

| Paso | Actor        | Acción                                                                 |
|------|--------------|------------------------------------------------------------------------|
| 1    | Usuario      | Accede al sistema Galaxy One                                           |
| 2    | Interfaz     | Muestra pantalla de bienvenida                                         |
| 3    | Usuario      | Clic en “Iniciar sesión con Microsoft”                                |
| 4    | Interfaz     | Redirige a pantalla de login de MS Entra ID                           |
| 5    | Usuario      | Ingresa correo electrónico                                             |
| 6    | MS Entra ID  | Verifica formato y existencia del correo en BD                        |
| 7    | MS Entra ID  | Si válido, solicita contraseña                                         |
| 8    | Usuario      | Ingresa contraseña                                                     |
| 9    | MS Entra ID  | Verifica credenciales en BD                                            |
| 10   | MS Entra ID  | Si válidas, envía código MFA                                           |
| 11   | Usuario      | Ingresa código MFA                                                     |
| 12   | MS Entra ID  | Verifica código                                                        |
| 13   | Usuario      | Clic en “Sí” o “No, no soy yo”                                         |
| 14   | MS Entra ID  | Si “Sí” y código correcto → acceso permitido                          |
| 15   | MS Entra ID  | Si “No” o código incorrecto → acceso denegado                         |
| 16   | Interfaz     | Muestra dashboard según rol o mensaje de error                        |
| 17   | Sistema      | Registra intento en BD con fecha, IP, resultado, error y georreferencia |
