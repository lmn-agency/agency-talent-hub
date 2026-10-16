# Arquitectura de Datos y Métricas Consolidadas | LMN Agency

Este documento detalla la arquitectura técnica para la gestión de datos, métricas sociales (Instagram y TikTok) y cálculo automático de estadísticas consolidadas en el portal **LMN Agency Talent Hub**, alojado de forma estática en **GitHub Pages**.

---

## 1. El Reto de Rendimiento en GitHub Pages

GitHub Pages es un entorno de alojamiento puramente estático (sin servidor backend de Node.js, Python o PHP ejecutándose al vuelo).

* **Enfoque ingenuo (No recomendado):** Llamar a las APIs de Instagram y TikTok directamente desde el navegador de cada usuario con JavaScript.
  * **Problemas:** Expone credenciales secretas en el código público, provoca bloqueos por *Rate Limiting* (límite de peticiones de Meta/TikTok) y congela la pantalla del visitante de 1 a 3 segundos mientras espera respuesta de servidores externos.
* **Enfoque Profesional LMN (JAMstack Serverless):** Desacoplar la **recolección de datos** de la **visualización**.
  * La recolección corre en segundo plano vía automatización (**GitHub Actions**).
  * Los datos se consolidan en un único archivo de verdad: [`assets/data/talents-data.json`](../assets/data/talents-data.json).
  * El visitante descarga el archivo desde el CDN de GitHub en menos de **15 milisegundos**, garantizando una carga instantánea y fluida.

---

## 2. Flujo de Datos (Arquitectura)

```
┌────────────────────────────────────────────────────────┐
│  SERVICIOS EXTERNOS (APIs Oficiales)                  │
│  • Meta Graph API (Instagram Creador / Business)       │
│  • TikTok Display API / Scraper Seguro                 │
└──────────────────────────┬─────────────────────────────┘
                           │ Ejecución programada (CRON cada 6h/12h)
                           ▼
┌────────────────────────────────────────────────────────┐
│  GITHUB ACTIONS RUNNER (Entorno Privado y Seguro)     │
│  • Lee Secrets cifrados (Tokens de Meta y TikTok)     │
│  • Consulta métricas de cada talento                   │
│  • Valida respuestas y calcula sumatorias              │
│  • Actualiza: assets/data/talents-data.json            │
└──────────────────────────┬─────────────────────────────┘
                           │ Commit y Push automático
                           ▼
┌────────────────────────────────────────────────────────┐
│  GITHUB PAGES CDN (Entrega Estática Instantánea)       │
│                                                        │
│  ┌───────────────────────┐  ┌────────────────────────┐ │
│  │     index.html        │  │  EPK Talento Individual│ │
│  │ (Roster + Métricas    │  │  (Stephanie Almeida    │ │
│  │  Consolidadas)        │  │   Ficha de Casting)    │ │
│  └───────────────────────┘  └────────────────────────┘ │
└────────────────────────────────────────────────────────┘
```

---

## 3. Estructura de la Fuente Única de Verdad: `talents-data.json`

Ubicado en [`assets/data/talents-data.json`](../assets/data/talents-data.json), este archivo contiene tanto los talentos reales como los activos en desarrollo:

```json
{
  "updatedAt": "2026-10-16T14:50:00Z",
  "collective": {
    "exclusivityPercentage": 100
  },
  "talents": [
    {
      "id": "stephanie-almeida",
      "name": "Stephanie Almeida",
      "status": "active",
      "isReal": true,
      "slug": "stephanie-almeida-creador-1",
      "profilePath": "./talents/stephanie-almeida-creador-1/index.html",
      "categories": ["actuacion", "lifestyle", "comedia"],
      "networks": {
        "instagram": {
          "handle": "stephaniesk8",
          "url": "https://www.instagram.com/stephaniesk8",
          "followers": 54500,
          "engagementRate": 11.1
        },
        "tiktok": {
          "handle": "stephanie_style8",
          "url": "https://www.tiktok.com/@stephanie_style8",
          "followers": 21600,
          "likes": 340000
        }
      },
      "monthlyReach": 2500000
    }
  ]
}
```

---

## 4. Cálculo Dinámico de Métricas Consolidadas

En lugar de escribir números fijos en el HTML de la página principal (`index.html`), el script carga el JSON y computa en tiempo real las sumatorias matemáticas exactas:

### Fórmulas Matemáticas:
1. **Audiencia Total Consolidada:**
   $$\text{Audiencia Total} = \sum (\text{Instagram Followers} + \text{TikTok Followers})$$
2. **Alcance Mensual Combinado:**
   $$\text{Alcance Total} = \sum (\text{Monthly Reach / Video Views})$$
3. **Tasa de Exclusividad:**
   $$\text{Exclusividad} = \frac{\text{Talentos con Contrato Exclusivo}}{\text{Total de Talentos}} \times 100\%$$

### Ventajas:
* **Coherencia Matemática 100%:** Si Stephanie suma 5,000 seguidores en Instagram, la métrica global de la agencia sube en 5,000 en el siguiente ciclo.
* **Separación de Talentos Reales vs Proyección:** Se puede configurar el script para sumar únicamente los talentos con flag `"isReal": true` o incluir todo el roster activo.
* **Animación Numérica (Odometer):** Al entrar a la web, los números inician en 0 y suben fluidamente en 1.5 segundos con aceleración cúbica (*easeOutExpo*), transmitiendo dinamismo y modernidad.
