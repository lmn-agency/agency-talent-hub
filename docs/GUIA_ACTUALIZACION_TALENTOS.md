# Guía de Edición y Actualización de Talentos | LMN Agency

Este documento explica cómo actualizar la información física, fotos, reels y métricas de los talentos, resolviendo las dudas comunes sobre el funcionamiento del código interno.

---

## 1. El Misterio de los "Parámetros Físicos / Medidas que no cambiaban"

### ¿Qué ocurría?
En la página de Stephanie Almeida (`talents/stephanie-almeida-creador-1/index.html`), el archivo contiene **dos lugares** donde aparecen las medidas:
1. **El marcado HTML estático** (alrededor de las líneas 490 - 512):
   ```html
   <dd id="spec-height" class="...">1.65 m</dd>
   <dd id="spec-age-range" class="...">28 - 35 años</dd>
   ```
2. **El objeto de configuración JavaScript `PROFILE_CONFIG`** (alrededor de la línea 882):
   ```javascript
   casting: {
     height: "1.68 m",      // <--- Este valor sobreescribía el HTML al cargar la página
     ageRange: "22 - 28 años",
     eyes: "Café oscuro",
     ...
   }
   ```

### ¿Por qué está diseñado así?
El perfil utiliza una función llamada `renderCastingData()` que se ejecuta cuando el navegador termina de leer el documento (`DOMContentLoaded`). Esta función lee `PROFILE_CONFIG.casting` y **rellena dinámicamente los campos** y el botón de *"Copiar Ficha de Casting al Portapapeles"*. 

Por lo tanto, al cambiar solo el HTML, JavaScript volvía a escribir encima los datos antiguos de `PROFILE_CONFIG`.

### Solución aplicada:
Se sincronizó `PROFILE_CONFIG.casting` con los valores oficiales actualizados:
* **Estatura:** `1.65 m`
* **Rango de edad:** `28 - 35 años`
* **Ojos:** `Verde`
* **Piel:** `Blanca`
* **Calzado:** `23.5 MX / 6.5 US`

> **Regla de oro:** Al actualizar medidas o casting de cualquier talento, asegúrate de actualizar el bloque `PROFILE_CONFIG.casting` dentro del `<script>` al final de su archivo `index.html`.

---

## 2. Diferencia entre Stephanie (Real) y los Otros Talentos (Ficticios / Provisionales)

En el estado actual de la plataforma:

| Apartado | Stephanie Almeida (Talento Real #1) | Otros 5 Talentos (Claudia, Ana, Rubén, etc.) |
| :--- | :--- | :--- |
| **Página individual (EPK)** | Tiene carpeta propia con página completa: `talents/stephanie-almeida-creador-1/index.html` | No tienen carpeta individual aún; sus tarjetas en `index.html` apuntan a `#roster`. |
| **Fotografías** | WebP originales de alta resolución en `assets/images/stephanie/` | Fotos provisionales de Unsplash. |
| **Showreels 9:16** | 4 reels reales catalogados con vistas y duraciones exactas | No aplican por el momento. |
| **Métricas** | Reales: 54.5K Instagram + 21.6K TikTok = 76.1K | Ficticias / Provisionales para maquetación. |

---

## 3. Cómo Dar de Alta a un Nuevo Talento Real

Cuando se firme o incorpore un nuevo talento a la agencia:

### Paso 1: Duplicar la plantilla de perfil
1. Crear una carpeta dentro de `talents/` siguiendo la convención de URLs limpias:
   `talents/nombre-apellido-creador-X/` (ej. `talents/claudia-zepeda-creador-2/`).
2. Copiar el `index.html` de Stephanie como base.

### Paso 2: Subir sus fotografías y assets
* Crear una carpeta en `assets/images/claudia/` con:
  * `main.webp` (Foto vertical de cartelera 3:4).
  * `acting-headshot.webp`, `middle-shot.webp`, `full-body.webp`, `editorial.webp`.

### Paso 3: Ajustar `PROFILE_CONFIG` en su `index.html`
Modificar el objeto con sus datos reales:
* Nombre, slug y títulos artísticos.
* Redes sociales y enlaces.
* Bloque `casting`: Estatura, tallas, idiomas, acentos y habilidades.
* Bloque `reels`: Enlaces a videos verticales y vistas.

### Paso 4: Registrar al talento en `assets/data/talents-data.json`
Agregar su registro en la lista de talentos para que sus seguidores se sumen automáticamente a la métrica global del colectivo.

### Paso 5: Activar su tarjeta en el Roster de `index.html`
Cambiar el enlace de su tarjeta para que apunte a su nueva ruta:
`href="./talents/claudia-zepeda-creador-2/index.html"`.

---

## 4. Modificación Manual de Métricas (Variables Locales)

Si la agencia **aún no tiene configurado el script `sync_metrics.py`** o los tokens de API de Meta/TikTok, las métricas de cualquier talento se pueden editar directamente mediante variables JavaScript dentro de su archivo `index.html`:

### Ubicación: En `talents/stephanie-almeida-creador-1/index.html` (línea ~807)

```javascript
// 1. Seguidores en botones de redes sociales:
socials: [
  {
    platform: "tiktok",
    name: "TikTok",
    handle: "@stephanie_style8",
    url: "https://www.tiktok.com/@stephanie_style8",
    followers: "21.6K", // <--- Modificar aquí directamente
    highlight: "Lifestyle"
  },
  {
    platform: "instagram",
    name: "Instagram",
    handle: "@stephaniesk8",
    url: "https://www.instagram.com/stephaniesk8",
    followers: "54.5K", // <--- Modificar aquí directamente
    highlight: "Actuación & Moda"
  }
],

// 2. Tarjetas de métricas del Hero:
metrics: {
  totalFollowers: "76.1K+", // <--- Modificar Audiencia Total
  monthlyViews: "34.4K",     // <--- Modificar Promedio Mensual
  engagementRate: "11.1%",   // <--- Modificar Engagement Rate
  primaryPlatforms: "TikTok • Instagram • YouTube Shorts"
}
```

Al modificar cualquiera de estos valores en el archivo, se reflejarán instantáneamente en la página sin tocar la estructura HTML ni depender de scripts externos.

