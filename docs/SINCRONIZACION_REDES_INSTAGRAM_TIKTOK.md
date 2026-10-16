# Guía de Sincronización Automática: Instagram & TikTok | LMN Agency

Esta guía explica paso a paso cómo conectar las estadísticas oficiales de los talentos (Instagram y TikTok) a través de un flujo automatizado en **GitHub Actions**, sin costos adicionales y sin exponer claves en el código fuente.

---

## 1. Conexión de Instagram (Meta Graph API)

Para leer seguidores, engagement e impresiones de una cuenta de Instagram de forma legal y estable:

### Requisitos Previos:
1. **Tipo de Cuenta:** La cuenta del talento (ej. `@stephaniesk8`) debe ser **Cuenta Profesional** (modo *Creador* o *Empresa*). Las cuentas personales no tienen API de métricas.
2. **Página de Facebook Vinculada:** Instagram exige que la cuenta esté vinculada a una Fan Page de Facebook (puede ser privada o administrada por la agencia).
3. **App en Meta for Developers:**
   * Entrar a [developers.facebook.com](https://developers.facebook.com/).
   * Crear una App tipo "Negocio" o "Consumidor".
   * Agregar el producto **Instagram Graph API**.
   * Permisos necesarios: `instagram_basic`, `pages_show_list`, `instagram_manage_insights`.

### Token de Acceso de Larga Duración:
* Meta entrega tokens temporales (1 hora) que se pueden intercambiar por un **Long-Lived Access Token** (60 días).
* El script automatizado puede auto-renovar este token antes de su vencimiento llamando al endpoint:
  ```http
  GET https://graph.facebook.com/v19.0/oauth/access_token?grant_type=fb_exchange_token&client_id={app-id}&client_secret={app-secret}&fb_exchange_token={token-actual}
  ```

### Endpoints Clave:
* **Seguidores y Biografía:**
  `GET https://graph.facebook.com/v19.0/{instagram-user-id}?fields=followers_count,media_count,profile_picture_url`
* **Engagement de Publicaciones Recientes:**
  `GET https://graph.facebook.com/v19.0/{instagram-user-id}/media?fields=like_count,comments_count,media_type`

---

## 2. Conexión de TikTok

Para TikTok existen dos vías de integración:

### Vía A: TikTok Display API (Oficial)
* A través del portal [developers.tiktok.com](https://developers.tiktok.com/).
* El talento autoriza a la app mediante OAuth 2.0.
* Endpoint: `https://open.tiktokapis.com/v2/user/info/?fields=follower_count,likes_count,video_count`
* **Ventaja:** Conexión oficial y métricas exactas.
* **Desventaja:** Proceso de revisión y aprobación de TikTok.

### Vía B: Scraper Headless en GitHub Action (Sin Tokens ni Aprobaciones)
* Como el runner de GitHub Actions es una máquina virtual completa con Linux/Ubuntu, puede ejecutar un script ligero de Python (`yt-dlp` o `httpx`) que extrae la información pública del perfil (seguidores y likes) directamente de la cabecera JSON pública que TikTok expone a navegadores.
* **Ventaja:** No requiere crear apps de desarrollador ni pedir tokens a los talentos.
* **Desventaja:** Si TikTok cambia la estructura HTML pública, requiere ajustar el extractor.

---

## 3. Automatización en GitHub Actions (`.github/workflows/sync-stats.yml`)

El siguiente flujo corre de forma programada en la infraestructura de GitHub:

```yaml
name: Sincronizar Métricas de Instagram y TikTok

on:
  schedule:
    # Se ejecuta todos los días a las 04:00 AM UTC (Horario nocturno de bajo tráfico)
    - cron: '0 4 * * *'
  workflow_dispatch: # Permite ejecutarlo manualmente con un clic en GitHub

jobs:
  update-stats:
    runs-on: ubuntu-latest
    steps:
      - name: Descargar repositorio
        uses: actions/checkout@v4

      - name: Configurar Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Instalar dependencias
        run: |
          pip install requests

      - name: Ejecutar script de actualización de métricas
        env:
          META_ACCESS_TOKEN: ${{ secrets.META_ACCESS_TOKEN }}
          TIKTOK_APP_TOKEN: ${{ secrets.TIKTOK_APP_TOKEN }}
        run: |
          python scripts/sync_metrics.py

      - name: Guardar cambios si hay actualizaciones
        run: |
          git config --global user.name 'LMN Metrics Bot'
          git config --global user.email 'bot@lmnagency.com'
          git add assets/data/talents-data.json
          git diff --quiet && git diff --staged --quiet || (git commit -m "chore(metrics): auto-update social metrics [skip ci]" && git push)
```

---

## 4. Configuración de Secretos en GitHub (GitHub Secrets)

Para que el script pueda acceder a las APIs sin exponer ninguna clave:
1. En el repositorio de GitHub, ir a **Settings** → **Secrets and variables** → **Actions**.
2. Hacer clic en **New repository secret**.
3. Agregar:
   * `META_ACCESS_TOKEN`: Token de larga duración de Meta.
   * `TIKTOK_APP_TOKEN` (opcional si se usa la vía oficial).

---

## 5. Estrategia de Resiliencia (Fail-Safe)

Para asegurar que la web de LMN Agency **nunca muestre errores o números en cero**:
1. El script lee el archivo `assets/data/talents-data.json` actual.
2. Si la API de Instagram o TikTok arroja error de red o timeout, el script **mantiene los valores del día anterior**.
3. Se genera un log del error visible en la pestaña "Actions" del repositorio, alertando a la agencia sin afectar a los usuarios en la web pública.
