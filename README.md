# Conflict Tracker-Desplegament fullstack

## 1. URL publica
- Frontend →  https://conflicttracker-frontend2026.vercel.app

## 2. Esquema de la arquitectura utilizada
Descripcion:
- Frontend: Vue 3 + Vite, desplegado en Vercel.
- Backend: Spring Boot (REST API) desplegado en Railway.
- Base de datos: PostgreSQL en Supabase (acceso desde Railway).
- Comunicación:
  - Frontend → (HTTPS) → Vercel (hosting). Peticiones a '/api/*' proxeadas a Railway mediante ‘vercel.json’ ó llamadas directas a ‘VITE_API_URL’.
  - Backend → (JDBC) → Supabase Postgres.

## Diagrama arquitectura
Frontend (Vercel)
    │
    ├─ user browser ──> https://conflicttracker-frontend2026.vercel.app
                         │
                         └─ /api/*  (rewrites proxy)  ──> Backend REST (Railway)
                                                     └─ JDBC ──> Supabase (Postgres)

## 3. Instrucciones para configurar las variables de entorno

Listan las variables necesarias para que el backend (Spring Boot) y el frontend (Vite) funcionen correctamente en producción y en desarrollo local.

### Backend (Railway / Spring Boot)

Variables requeridas
-FRONTEND_URL
  -ejemplo:https://conflicttracker-frontend2026.vercel.app
- SPRING_DATASOURCE_URL  
  - Ejemplo: 'jdbc:postgresql://db.abcxyz.supabase.co:5432/fullstack_db'
- SPRING_DATASOURCE_USERNAME  
  - Ejemplo: 'postgres'
- SPRING_DATASOURCE_PASSWORD  
  - Ejemplo: 'Contraseña123!'
- SPRING_PROFILES_ACTIVE  
  - Valor recomendado: 'prod'

Cómo añadirlas en Railway
1. Entra al proyecto en Railway.
2. Ve a la sección "Variables".
3. Añade cada variable (Name / Value).
4. Guarda y redeploy

### Frontend (Vercel / Vite)

Variables
-Usando variable directa:
  - VITE_API_URL 
    - Valor: 'https://conflict-tracker-production-aa6b.up.railway.app'  
    - Uso: el frontend usará 'import.meta.env.VITE_API_URL' como base para las peticiones.
  
## 4. Lista de modificaciones en Backend y Frontend

### Modificaciones añadidas:

1.Añadimo src/main/resources/application-prod.yaml
  versión YAML para producción que lee las mismas variables de entorno.
 -Motivo: separar configuración de prod y usar variables seguras en el host (Railway).

2. Añadido en la raíz: vercel.json
 añadir rewrite/proxy para que las llamadas /api/* en Vercel lleguen a Railway
-Motivo: en producción en Vercel las rutas relativas necesitan proxear al backend,con esto no hace falta cambiar el frontend.

3.Añadimos src/main/resources/application.properties
Lee la configuración de la BD y credenciales desde variables de entorno (SPRING_DATASOURCE_*). Se añade también puerto por defecto y configuración JPA.
-Motivo: evitar credenciales en el código, permitir que el host (Railway) suministre las variables seguras y facilitar el uso de perfiles (prod/dev).


### Errores y soluciones

1.Error: La UI aparece pero "No hay conflictos" / lista vacía
-Síntoma: en DevTools Network, la petición a /api/v1/conflicts devuelve 404/502 o HTML de la página en lugar de JSON.
-Causa típica: Vercel no reenvía /api/* al backend y frontend usa rutas relativas.
-Solución: añadir vercel.json con rewrite que proxee /api/* a la URL del backend, o configurar VITE_API_URL, luego redeploy.

2.Error: CORS blocked (en consola del navegador)
-Síntoma: Error CORS que indica Origin not allowed al hacer fetch.
-Causa: Backend no permite el origen del frontend
-Solución : usar proxy en Vercel (vercel.json) para evitar CORS.

3.Error: Conexión a DB fallida (PSQLException / authentication failed)
-Síntoma: excepciones en logs de Railway al arrancar (auth error, connection refused).
-Causa: SPRING_DATASOURCE_URL/USERNAME/PASSWORD incorrectos o DB no accesible.
-Solución:
Verificar SPRING_DATASOURCE_URL tiene formato JDBC correcto: jdbc:postgresql://host:port/database?sslmode=require
Verificar username/password 
Si cambias password en Supabase, actualizar en Railway y redeploy.


