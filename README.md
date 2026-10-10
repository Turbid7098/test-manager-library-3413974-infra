# test-manager-library-3413974-infra

> **Prueba técnica · Ficha ADSO 3413974**
> Horario: de **9:15 a. m. a 3:00 p. m.** (15:00). Después, **feria de presentación** de cada proyecto.

Este repositorio es la **infraestructura** de *Manager Library*: contenedores, red, volúmenes y el motor de base de datos vacío. **Empieza vacío a propósito**: se construye en el fork de cada aprendiz.

## Instrucciones

Cada aprendiz debe **crear el fork** de los seis repositorios del proyecto y **resolver el proyecto
con el spec planteado**.

1. Hacer fork, a su cuenta de GitHub, de cada repositorio de la tabla del final.
2. Leer el spec en [`test-manager-library-3413974-docs`](https://github.com/code-sena/test-manager-library-3413974-docs).
   Se entrega en dos versiones: `spec-python/` y `spec-.net/`.
3. Desarrollar en los forks.

## El reto se desarrolla con React y PHP (Laravel)

El spec está escrito para Python y para .NET, pero el reto **no** se hace en esos lenguajes:

| Capa | Tecnología del reto |
|---|---|
| Frontend | React |
| Backend | PHP con Laravel |

Lo que el spec define sobre el negocio —historias, criterios de aceptación, reglas, contrato de la
API, modelo de datos— se respeta. Lo que define sobre la tecnología se traduce a React y Laravel.

## La prueba no consiste en escribir el código

El propósito principal es ver la **capacidad de desempeño con SDD** (*Spec-Driven Development*,
desarrollo guiado por especificación): cómo se lee, se interpreta y se aplica una especificación
para llevarla a un stack distinto. El código es el medio, no el fin.

## Feria de presentación

Al cerrar la prueba (3:00 p. m.) cada aprendiz **presenta su proyecto** al grupo: qué construyó del spec, qué dejó fuera y por qué,
y una demostración corta de lo que funciona (la aplicación y el sitio público). Lo que se presenta es lo que quedó **subido a los forks
antes de las 3:00 p. m.**: no se presenta nada que no esté en GitHub.

## Los seis repositorios

| Repositorio | Qué va ahí |
|---|---|
| [`test-manager-library-3413974-docs`](https://github.com/code-sena/test-manager-library-3413974-docs) | El spec: `spec-python/` y `spec-.net/` |
| [`test-manager-library-3413974-api`](https://github.com/code-sena/test-manager-library-3413974-api) | Backend en PHP (Laravel) |
| [`test-manager-library-3413974-app`](https://github.com/code-sena/test-manager-library-3413974-app) | Frontend en React |
| [`test-manager-library-3413974-page`](https://github.com/code-sena/test-manager-library-3413974-page) | Sitio público estático de presentación |
| [`test-manager-library-3413974-infra`](https://github.com/code-sena/test-manager-library-3413974-infra) | Contenedores, red, volúmenes y motor de base de datos vacío |
| [`test-manager-library-3413974-tool`](https://github.com/code-sena/test-manager-library-3413974-tool) | Utilidades: sembrador de datos de demostración |

## Ejecución local

Requisitos: Docker Desktop con Compose y los seis repositorios clonados dentro de una misma carpeta.

1. Desde este directorio, crea la configuración local ignorada por Git y define valores propios para la base de datos y el administrador:

   ```powershell
   Copy-Item .env.example .env
   notepad .env
   ```

   Reemplaza todos los valores `replace-with-...`; genera `APP_KEY` como `base64:` seguido de la codificación Base64 de 32 bytes aleatorios. No publiques `.env`, contraseñas, hashes ni tokens.

2. Construye y levanta PostgreSQL, Laravel y Vite:

   ```powershell
   docker compose up --build -d
   ```

3. Espera el healthcheck de PostgreSQL y aplica el esquema propiedad de las migraciones Laravel:

   ```powershell
   docker compose exec api php artisan migrate --force
   docker compose exec api php artisan app:create-admin
   ```

   El comando de administrador es idempotente: un administrador existente no se duplica y un nombre ocupado por otro rol produce error.

4. Comprueba los endpoints públicos:

   ```powershell
   Invoke-RestMethod http://localhost:8000/api/v1/health
   ```

   Health debe devolver `status: ok`. El login `POST http://localhost:8000/api/v1/auth/login` acepta `username` y `password` como JSON y devuelve un JWT Bearer con vigencia de ocho horas. Pruébalo con el administrador configurado localmente; no publiques las credenciales ni el token. P2 sigue parcial hasta probar el arranque desde un volumen PostgreSQL vacío.

La base arranca sin tablas de negocio; estas aparecen al ejecutar las migraciones. Para detener los servicios sin borrar los datos, usa `docker compose down`. No uses `down -v` salvo que quieras eliminar deliberadamente el volumen de PostgreSQL.

El servicio API ejecuta `php artisan serve --no-reload` para conservar las variables de entorno definidas por Compose incluso cuando existe un `.env` local.
