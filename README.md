# Cuentas de Casa

App de gestión de finanzas familiares: ingresos, gastos fijos, gastos variables (con carga desde foto de factura), cuotas de tarjeta y categorías. Funciona en el celular y en la computadora como una app instalable (PWA).

## Archivos

- `index.html` — la app completa (HTML + CSS + JS, no necesita nada más)
- `manifest.json` — configuración para que se pueda instalar como app
- `sw.js` — service worker para que funcione sin conexión
- `icon-192.png`, `icon-512.png` — íconos de la app

## Cómo subirla (gratis, sin saber programar)

La forma más simple es **Netlify Drop**:

1. Entrá a https://app.netlify.com/drop
2. Arrastrá esta carpeta completa (`cuentas-app`) a la página.
3. En unos segundos te da una URL pública (algo como `https://nombre-al-azar.netlify.app`).
4. Abrí esa URL desde el celular y la compu.

Otra opción es **GitHub Pages**:

1. Creá un repositorio nuevo en GitHub (puede ser privado).
2. Subí estos archivos al repositorio (botón "Add file" → "Upload files").
3. Entrá a *Settings → Pages*, elegí la rama `main` y la carpeta raíz (`/`).
4. GitHub te va a dar una URL tipo `https://tuusuario.github.io/tu-repo/`.

## Instalarla como app

- **Celular (Android/iPhone)**: abrí la URL en Chrome o Safari, tocá el menú (⋮ o el ícono de compartir) y elegí "Agregar a pantalla de inicio" / "Añadir a inicio". Va a quedar como un ícono más, abre a pantalla completa.
- **Compu (Chrome/Edge)**: abrí la URL, en la barra de direcciones aparece un ícono de "Instalar app" (o menú → "Instalar Cuentas de Casa").

## Dónde se guardan los datos

Todo se guarda **en el navegador de cada dispositivo** (no en internet, no en ningún servidor). Eso significa:

- Es privado: nadie más ve tus datos.
- Si usás la app en el celular **y** en la compu, son dos "bases" separadas. Para mantenerlas iguales:
  1. En el dispositivo donde cargaste los datos, ir a **Ajustes → Exportar datos**. Se descarga un archivo `.json`.
  2. Pasar ese archivo al otro dispositivo (por mail, WhatsApp a vos mismo, Drive, etc.).
  3. En el otro dispositivo, **Ajustes → Importar datos** y elegir ese archivo.
- Si borrás el navegador / los datos del sitio, perdés la información. Por eso conviene exportar de vez en cuando como respaldo.

## Escanear facturas con IA (opcional)

Para que el botón "📷 Cargar desde factura" funcione, necesitás tu propia clave de la API de Anthropic (Claude):

1. Entrá a https://console.anthropic.com → "API Keys" → crear una clave nueva.
2. En la app, ir a **Ajustes** y pegarla en "Escaneo de facturas con IA" → Guardar.
3. La clave queda guardada solo en ese navegador/dispositivo.

Esto tiene un costo muy bajo por uso (la API de Anthropic se cobra según el uso; analizar una foto de ticket cuesta centavos de dólar). Si no querés usar esta función, simplemente no configurés la clave y cargá los gastos a mano.

## Sincronizar entre dispositivos (Supabase)

Para que el celular y la compu compartan los mismos datos automáticamente, conectá la app a una base de datos gratuita en Supabase:

1. Entrá a https://supabase.com → "Start your project" → creá una cuenta (podés usar GitHub) → "New project".
   - Ponele un nombre (ej. `cuentas-de-casa`), una contraseña para la base (guardala, no la vas a necesitar de nuevo en esta app) y elegí una región cercana.
2. Esperá un par de minutos a que se cree el proyecto.
3. Ir a **SQL Editor** (ícono de la izquierda) → "New query" y pegar esto, después "Run":

   ```sql
   create table public.cuentas (
     id text primary key,
     data jsonb not null,
     updated_at timestamptz not null default now()
   );

   alter table public.cuentas enable row level security;

   create policy "allow all" on public.cuentas
     for all using (true) with check (true);
   ```

4. Ir a **Settings → API**. Ahí vas a ver:
   - **Project URL** → algo como `https://abcdefgh.supabase.co`
   - **anon public** key → una clave larga que empieza con `eyJ...`
5. En la app, ir a **Ajustes → Sincronizar entre dispositivos** y completar:
   - URL del proyecto
   - anon public key
   - Código de sincronización: lo elegís vos, por ejemplo `CASA-JP-2026`. Tiene que ser **igual** en todos los dispositivos que quieras sincronizar.
6. Tocá "Guardar y conectar". En el otro dispositivo, repetí el paso 5 con los **mismos** datos (URL, key y código).

A partir de ahí, cada vez que cargues un gasto la app lo guarda en la nube, y al abrir la app en otro dispositivo trae los datos más recientes. También podés tocar "Sincronizar ahora" para forzarlo.

**Importante sobre seguridad**: con esta configuración, cualquiera que tenga la URL, la clave "anon" y tu código de sincronización podría leer o modificar esos datos (la política de la base permite todo). Para uso familiar normal está bien, pero no compartas esos tres datos ni los dejes en un repositorio público de GitHub. Si en algún momento querés algo más seguro (login con usuario y contraseña), se puede agregar más adelante.



Podés editar `index.html` con cualquier editor de texto:

- Categorías y grupos por defecto: buscar `SEED_CATEGORIES` y `SEED_GROUPS`.
- Colores: al principio del archivo, dentro de `<style>`, en `:root { ... }`.
