# StreamFlix PRO — Estructura Cloudflare

## Estructura del proyecto

```
streamflix-cf/
├── index.html                  # Página principal (shell de la app)
├── css/
│   ├── base.css                 # Variables, reset
│   ├── layout.css                # Header, hero, grids, filas
│   ├── components.css            # Cards, botones, modales, buscador
│   └── player.css                # Reproductor de video
├── js/
│   ├── app.js                    # Punto de entrada, routing entre vistas
│   ├── api/
│   │   ├── xtream.js              # Cliente directo al panel Xtream Codes
│   │   └── backend.js             # Cliente hacia las Pages Functions (D1)
│   ├── modules/
│   │   ├── auth.js                # Login/registro de usuarios de la app
│   │   ├── profiles.js            # Perfiles tipo Netflix
│   │   ├── catalog.js             # Hero, shelves, grids de películas/series
│   │   ├── tvlive.js              # Canales en vivo + EPG
│   │   ├── search.js              # Búsqueda predictiva
│   │   ├── player.js              # Reproductor universal (HLS.js)
│   │   └── favorites.js           # "Mi Lista"
│   └── utils/
│       ├── device.js              # Detección TV/desktop/tablet/móvil
│       └── cache.js               # IndexedDB (progreso, índice de búsqueda)
├── functions/                   # Cloudflare Pages Functions (API serverless)
│   ├── _utils/session.js
│   └── api/
│       ├── auth/{login,register,me}.js
│       ├── profiles.js
│       ├── favorites/{index,[id]}.js
│       ├── progress/index.js
│       └── epg/[channelId].js
├── migrations/0001_init.sql     # Esquema D1
├── public/                      # manifest.json, sw.js, icons/
├── wrangler.toml
└── package.json
```

## Por qué esta arquitectura

- **Frontend estático → Xtream directo**: el navegador llama directo al panel Xtream Codes (`js/api/xtream.js`) para catálogo, EPG y URLs de streaming. Es más rápido y no gasta cuota de tus Functions.
- **Pages Functions → D1**: todo lo que es "tu" dato (usuarios, perfiles, favoritos, progreso) pasa por `functions/api/*`, que habla con D1. Nunca expone credenciales Xtream al hacer esto.
- **D1**: base SQL gestionada por Cloudflare, gratis hasta cuotas generosas, ideal para relaciones usuario→perfiles→favoritos/progreso.

## Pasos de despliegue

1. **Instalar Wrangler y autenticarte**
   ```bash
   npm install
   npx wrangler login
   ```

2. **Crear la base D1**
   ```bash
   npm run db:create
   ```
   Copia el `database_id` que te devuelve y pégalo en `wrangler.toml`.

3. **Aplicar el esquema**
   ```bash
   npm run db:migrate:remote
   ```

4. **Configurar variables de entorno**
   En el dashboard de Cloudflare Pages → tu proyecto → Settings → Environment Variables, o directamente en `wrangler.toml` (`[vars]`), define:
   - `XTREAM_BASE_URL`
   - `XTREAM_USER`
   - `XTREAM_PASS`

   También edita `js/modules/catalog.js` con tus credenciales Xtream reales (o mejor, monta un flujo de login que las guarde por usuario en D1 en vez de dejarlas fijas en el código).

5. **Probar en local**
   ```bash
   npm run dev
   ```

6. **Desplegar**
   ```bash
   npm run deploy
   ```
   O conecta el repo de GitHub/GitLab directamente en el dashboard de Cloudflare Pages para despliegue automático en cada push.

## Siguientes pasos sugeridos

- Migrar aquí la lógica que ya tienes en tu archivo único (intro "Impacto", CatalogDiff/NotifCenter, PLAYER_ENGINE con fallback de formatos) dentro de los módulos correspondientes en `js/modules/`.
- Añadir manejo de credenciales Xtream por usuario (guardadas en D1, no hardcodeadas).
- Generar los íconos reales en `public/icons/` (192x192 y 512x512 como mínimo) para que el manifest PWA funcione.
