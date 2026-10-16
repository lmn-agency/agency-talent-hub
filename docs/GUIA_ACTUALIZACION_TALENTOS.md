# Guía Centralizada de Talentos y Métricas | LMN Agency

Este documento detalla la arquitectura de **Documento Único (Single Source of Truth)** implementada para gestionar a los talentos de LMN Agency sin necesidad de tocar código HTML, y explica la resolución de discrepancias en las métricas entre la portada y los perfiles individuales.

---

## 1. El Sistema de Documento Único (`talents-data.json`)

Para que el equipo de la agencia no tenga que editar archivos HTML de 1,400 líneas, **el 100% de la información de los talentos se administra desde un solo archivo central**:

📁 **Ruta del archivo:** [`assets/data/talents-data.json`](../assets/data/talents-data.json)

### ¿Cómo funciona la sincronización automática?
1. **La Portada (`index.html`):** Lee este JSON al cargar. Suma la audiencia total de todos los talentos, calcula el alcance mensual del colectivo y actualiza las tarjetas del Roster.
2. **El Perfil de Stephanie (`talents/stephanie-almeida-creador-1/index.html`):** Lee este mismo JSON al cargar. Si cambias su estatura a `1.65 m`, agregas *"Portugués Brasileño"*, subes un nuevo reel o actualizas sus seguidores, la página se re-renderiza automáticamente con esos datos.
3. **Respaldo Offline (Fail-Safe):** Si el sitio se abre localmente sin servidor web (mediante doble clic `file://`), el perfil mantiene sus valores internos para garantizar que nunca se vea vacío ni rompa la estética.

> **En resumen:** Ya no necesitas tocar ni una sola línea de HTML en `index.html` ni en el perfil de Stephanie. Solo editas `talents-data.json`.

---

## 2. ¿Por qué diferían las métricas entre el Index Principal y cada Talento?

Durante la revisión técnica se detectaron tres motivos exactos por los cuales los números no coincidían:

### Motivo A: Desfase entre el HTML estático maquetado y el JSON previo
En `index.html` las tarjetas tenían los valores oficiales del dossier/PDF, mientras que el JSON provisional tenía cifras aproximadas. **Ya quedaron igualadas al 100%:**

| Talento | En la Portada (`index.html`) | En `talents-data.json` (Anterior) | Estado Actual Sincronizado |
| :--- | :--- | :--- | :--- |
| **Stephanie Almeida** | IG: 54.5K • TK: 21.6K (76.1K+) | IG: 54.5K • TK: 22.1K (76.6K) | **Sincronizado:** 54.5K IG + 22.1K TK = **76.6K Total** |
| **Claudia Zepeda** | IG: 224K • TK: 222.3K (446K+) | IG: 186K • TK: 260K | **Sincronizado:** 224K IG + 222.3K TK = **446.3K Total** |
| **Ana Tena** | IG: 1.1M • TK: 7.3M (8.4M+) | IG: 1.4M • TK: 7.0M | **Sincronizado:** 1.1M IG + 7.3M TK = **8.4M Total** |
| **Rubén Saucedo** | IG: 34.4K • TK: 288K (322K+) | IG: 92K • TK: 230K | **Sincronizado:** 34.4K IG + 288K TK = **322.4K Total** |
| **Trinidad González**| IG: 88.9K • TK: 288K (376K+) | IG: 116K • TK: 260K | **Sincronizado:** 88.9K IG + 288K TK = **376.9K Total** |
| **Frida Jiser** | IG: 16K • TK: 109.4K (125K+) | IG: 45K • TK: 80K | **Sincronizado:** 16K IG + 109.4K TK = **125.4K Total** |

### Motivo B: La edición manual reciente de Stephanie
Cuando actualizaste TikTok de Stephanie a `22.1K` en el JSON, la sumatoria subió a `76.6K`, pero los textos del HTML seguían mostrando `21.6K` y `76.1K+`. Con la nueva conexión automática, el HTML ahora lee directamente los `22,100` y calcula `76.6K+` en ambos sitios.

### Motivo C: Concepto de "Alcance Mensual Colectivo" vs "Promedio Mensual de Stephanie"
* **En la Portada:** Se muestra **+60M Alcance Mensual**. Esto representa el **volumen total de reproducciones combinadas** de videos verticales de todos los 6 talentos de la agencia al mes.
* **En el Hero de Stephanie:** Se muestra **34.4K Promedio Mensual**. Esto se refiere al **promedio de reproducciones orgánicas por video individual** en su perfil personal de TikTok/Reels, no al tráfico global de la agencia.

---

## 3. Guía de Edición Campo por Campo en `talents-data.json`

Para actualizar a Stephanie (o a cualquier talento), abre [`assets/data/talents-data.json`](../assets/data/talents-data.json) y modifica el bloque correspondiente:

```json
{
  "id": "stephanie-almeida",
  "name": "Stephanie Almeida",
  
  // 1. Redes Sociales y Métricas (Se actualizan en Portada y Perfil)
  "networks": {
    "instagram": {
      "handle": "stephaniesk8",
      "followers": 54500,
      "engagementRate": 11.1
    },
    "tiktok": {
      "handle": "stephanie_style8",
      "followers": 22100,
      "likes": 88300
    }
  },

  // 2. Ficha Técnica de Casting (Se actualiza en su EPK)
  "casting": {
    "height": "1.65 m",
    "ageRange": "28 - 35 años",
    "eyes": "Verde",
    "hair": "Castaño oscuro",
    "skin": "Blanca",
    "shoes": "23.5 MX / 6.5 US",
    "languages": [
      { "name": "Portugués Brasileño", "level": "Nativo" },
      { "name": "Español", "level": "Avanzado" },
      { "name": "Inglés", "level": "Intermedio" }
    ],
    "accents": [
      "Español con acento extranjero",
      "Portugués Brasileño (nativo)"
    ],
    "skills": [
      "Actuación dramática",
      "Comedia situacional e improv",
      "Baile con ritmos latinos"
    ],
    "brands": [
      "Puma México",
      "Telcel",
      "Tena",
      "Blim"
    ]
  },

  // 3. Showreels de Video 9:16
  "reels": [
    {
      "id": 1,
      "title": "Miniso 'Encuentra tu momento'",
      "category": "Anuncio para Miniso",
      "views": "250K",
      "duration": "0:30",
      "videoPoster": "../../assets/images/stephanie/Portada Video Miniso.webp",
      "videoUrl": "https://pub-...r2.dev/Video Miniso.mp4"
    }
  ],

  // 4. Book Fotográfico
  "gallery": [
    {
      "id": "g1",
      "type": "Primer Plano (Acting Headshot)",
      "subtitle": "Enfoque en expresividad y mirada.",
      "url": "../../assets/images/stephanie/Stephanie Almeida dress.webp"
    }
  ]
}
```

Al guardar este archivo JSON, los cambios se reflejan inmediatamente en la web.
