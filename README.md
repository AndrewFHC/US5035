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

sequenceDiagram
    participant Usuario
    participant Interfaz
    participant MS Entra ID
    participant BD

    Usuario->>Interfaz: Accede a Galaxy One
    Interfaz->>Usuario: Muestra pantalla de bienvenida
    Usuario->>Interfaz: Clic en "Iniciar sesión con Microsoft"
    Interfaz->>MS Entra ID: Redirige a pantalla de login
    Usuario->>MS Entra ID: Ingresa correo electrónico
    MS Entra ID->>BD: Verifica formato y existencia del correo
    alt Correo válido
        MS Entra ID->>Usuario: Solicita contraseña
        Usuario->>MS Entra ID: Ingresa contraseña
        MS Entra ID->>BD: Verifica credenciales
        alt Credenciales válidas
            MS Entra ID->>Usuario: Envía código MFA
            Usuario->>MS Entra ID: Ingresa código MFA
            MS Entra ID->>BD: Verifica código
            alt Código correcto y clic en "Sí"
                MS Entra ID->>Interfaz: Autenticación exitosa
                Interfaz->>Usuario: Acceso a dashboard según rol
            else Código incorrecto o clic en "No"
                MS Entra ID->>Usuario: Acceso denegado
            end
        else Credenciales inválidas
            MS Entra ID->>Usuario: Muestra error de autenticación
        end
    else Correo inválido
        MS Entra ID->>Usuario: Muestra error de correo
    end

    Note over Interfaz,BD: Registrar intento de acceso con fecha, IP, resultado, error y georreferencia
