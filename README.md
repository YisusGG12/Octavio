# SIGMA-GTO
Sistema de Gestión y Reporte de Fugas de Agua en Guanajuato.

Proyecto completo: `frontend/` (React + Tailwind + Leaflet + OpenStreetMap) y
`backend/` (Node.js + Express + PostgreSQL/PostGIS + Nominatim API).

## Arranque rápido

```bash
# 1) Base de datos + API
cd backend
npm install
cp .env.example .env        # llena DATABASE_URL, JWT_SECRET
npm run migrate
npm run dev                 # http://localhost:4000

# 2) Frontend
cd ../frontend
npm install
npm run dev                 # http://localhost:5173
```

Cada carpeta tiene su propio `README.md`/comentarios con el detalle de cada
pieza. Revisa `backend/README.md` para el mapa completo de endpoints y las
correcciones aplicadas al script SQL original.

## Solución de Mapas: Leaflet y OpenStreetMap (OSM)

El sistema utiliza tecnologías libres y de código abierto para mapas y geolocalización:

- **Frontend** (`src/App.jsx`, componente `LeafletMapCanvas`): utiliza **Leaflet** y mosaicos de **OpenStreetMap** (OSM) para renderizar mapas interactivos, pins navegables y marcadores sin costo y sin necesidad de ninguna API Key.
- **Backend** (`src/utils/osmGeo.js`): utiliza la API libre **Nominatim** (OSM) para geocodificación y geocodificación inversa, convirtiendo direcciones en coordenadas `GEOMETRY(Point,4326)` en PostGIS.

## Estado del proyecto (esta sesión)

Se retomó el desarrollo siguiendo el orden de prioridad del prompt de
continuidad. Completado en esta sesión:

1. **Frontend↔backend real.** `seedReports` y las funciones `addReport`/
   `updateStatus` en memoria se eliminaron por completo. Cada vista
   (`AdminDashboard`, `TechnicianDashboard`, `CitizenDashboard`,
   `ReportFugaWizard`, `LoginView`, `RegisterView`, `LocationStepView`)
   ahora llama al backend real a través de `src/lib/api.js` (fetch + JWT),
   con estados de carga/error explícitos. El JWT se guarda en
   `localStorage` (`sigma_token` / `sigma_user`) porque este es un proyecto
   local con Vite, no un artifact embebido. El selector manual de rol en
   Login se quitó: el rol ahora siempre viene firmado en el JWT.
2. **Creación de técnicos.** Se agregó `POST /api/admin/technicians` y
   `GET /api/admin/technicians` (solo admin). Se optó porque un técnico NO
   se autoregistra: el admin los da de alta desde un formulario nuevo en
   la parte superior del panel ("Nuevo técnico"). La tabla de gestión del
   panel admin ahora también tiene una columna "Cuadrilla" para asignar el
   reporte a un técnico (`POST /api/admin/reports/:id/assign`).
3. **Subida real de imágenes.** El paso 1 del formulario "Reportar una
   Fuga" ahora sube cada foto con `POST /api/reports/:id/images` justo
   después de crear el reporte con `POST /api/reports`. Si una imagen
   falla, el reporte igual queda creado y se avisa cuál foto no subió.

Detalle técnico agregado en `backend/`: `optionalAuth` (nuevo middleware)
para que un reporte creado por un ciudadano con sesión activa quede
asociado a su cuenta (`ownerId`/`user_id`) sin dejar de aceptar reportes
anónimos, tal como pedía el brief.

### Pendiente (sin cambios, ver `PROMPT_CONTINUIDAD` original)
4. Notificaciones/avisos personalizados por zona.
5. Trazado de rutas óptimas con Google Maps API en el panel admin.
6. Pruebas end-to-end y despliegue (Docker / Render / Railway).

### Limitación conocida a señalar
El registro (`POST /api/auth/register`) solo guarda `home_location` cuando
el ciudadano usó el botón de GPS en la pantalla de Localización. Si en vez
de eso escribe una dirección a mano, esa dirección **no** se geocodifica
todavía en el registro (sí ocurre para los reportes de fugas, en
`googleGeo.js`, pero ese código no se reutilizó aquí). Es intencional para
no bloquear el flujo de registro sin key de Google configurada, pero es un
buen candidato a resolver junto con el punto 4 (avisos por zona), ya que
`home_location` es justamente lo que alimentaría esos avisos.
