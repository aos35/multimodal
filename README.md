
# SafeStep: Asistente Multimodal para Peatones

## 1. Introducción y Objetivos
**SafeStep** es un sistema de asistencia inteligente y accesible para peatones, especialmente pensado para personas con visión reducida. Utiliza visión artificial, procesamiento de lenguaje natural y síntesis de voz para aumentar la autonomía y seguridad en entornos urbanos complejos.

El sistema analiza vídeo en tiempo real (simulando una cámara wearable) y permite al usuario interactuar mediante comandos de voz o texto, filtrando la información relevante según la intención detectada.

**Objetivos principales:**
- Detectar peligros dinámicos (vehículos, peatones) y emitir alertas accesibles.
- Mejorar la percepción visual mediante smart zoom y superposiciones de alto contraste.
- Identificar pasos de cebra, semáforos y señales, estimando distancias relevantes.
- Ofrecer una interfaz multimodal (visual + audio) personalizable y eficiente.


## 2. Características Principales

### 🧠 Inteligencia Artificial y Procesamiento Multimodal
- **Detección y Tracking (YOLOv8 + ByteTrack):** Identifica y sigue vehículos, peatones, semáforos y señales. Calcula trayectorias y detecta peligros de colisión.
- **Comandos de Voz y NLP (Whisper + spaCy):** El usuario puede dar órdenes verbales o escritas (ej: *"Avísame solo de los semáforos"*). El sistema transcribe la voz y analiza la intención para filtrar las alertas y adaptar el modo de funcionamiento.
  - **OCR Híbrido:**
    - En los modos **SEÑALES** o **TODO**, el OCR se ejecuta periódicamente sobre regiones del frame y sobre cualquier objeto detectado como cartel o señal.
    - En otros modos, el OCR se activa automáticamente si se detecta un cartel relevante, permitiendo la lectura de textos útiles para el usuario.
    - El OCR puede ampliarse para nuevas clases de carteles añadiendo sus etiquetas en la lógica de detección.

### 👁️ Accesibilidad Visual Mejorada
- **Interfaz de Alto Contraste:** Elementos gráficos escalados para máxima visibilidad (textos grandes, bordes gruesos).
- **Radar de Seguridad Permanente:** Minimapa que muestra la posición relativa de los vehículos siempre, independientemente del modo activo.
- **Smart Zoom:** Lupa automática para semáforos y señales lejanas.
- **Detección de Paso de Cebra:** Análisis de contornos para identificar cruces peatonales seguros.

### 🎧 Audio Inteligente y Personalizable
- **Gestión de Fatiga de Alertas:** Sistema de colas con prioridades y cooldowns individuales.
- **Feedback Contextual:** Las alertas de audio se adaptan a la orden/intención del usuario.
- **Síntesis de Voz (gTTS):** Genera alertas de audio en español que avisan al usuario de los peligros y objetos deseados.


## 3. Modos de Operación

El sistema interpreta la intención del usuario (voz/texto) y activa uno de los siguientes modos:

| Modo         | Descripción                | Visualización principal         | Audio                |
|--------------|----------------------------|--------------------------------|----------------------|
| **TODO**     | Monitorización completa    | Todos los objetos              | Todas las alertas    |
| **SEMAFORO** | Foco en semáforos          | Solo semáforos + radar vehículos | Alertas de semáforo |
| **SEÑALES**  | Lectura de señales viales y carteles  | Señales, carteles + radar vehículos + OCR zonal | Lectura de texto |
| **PASO_CEBRA** | Búsqueda de cruces       | Detección de pasos + radar vehículos | Aviso de cruce seguro |
| **SOLO_PELIGRO** | Solo alertas críticas   | Vehículos en movimiento + radar vehículos + Textos | Solo peligros graves |

> **Nota de Seguridad:** Los vehículos siempre aparecen en el radar para garantizar la seguridad del usuario, independientemente del modo.


## 4. Estructura del Proyecto

```
multimodal/
├── SafeStep_Pipeline.ipynb   # Notebook principal con todo el pipeline
├── yolov8n.pt                # Modelo YOLOv8 preentrenado
├── README.md                 # Documentación del proyecto
├── videos/                   # Videos de entrada (sin procesar)
├── videos_safestep/          # Videos de salida (procesados con AR)
├── audios/                   # Audios de entrada para comandos de voz
├── audios_ordenes/           # Audios TTS generados (orden1.mp3, orden2.mp3...)
├── images/                   # Imágenes de entrada (OCR sin procesar)
├── images_safestep/          # Imágenes de salida (OCR)
└── temp/                     # Archivos temporales (limpieza automática)
```


## 5. Flujo de Ejecución (Pipeline)

1. **Entrada:** Video y orden del usuario (voz o texto).
2. **Transcripción y Análisis de Intención:** Whisper transcribe la voz y spaCy determina la intención y el modo de funcionamiento.
3. **Procesamiento Frame a Frame:**
  - **Preprocesamiento:** Mejora de contraste nocturno (CLAHE).
  - **Detección y Tracking:** YOLOv8 + ByteTrack.
  - **Validación Geométrica:** Filtrado por aspect ratio.
  - **Análisis de Color:** Lectura de semáforos en HSV.
  - **OCR Híbrido:** Lectura de señales según modo.
  - **Detección de Cruces:** Análisis de contornos para pasos de cebra.
  - **Gestión de Audio:** Decisión de alertas por prioridad, cooldown e intención.
  - **Renderizado AR:** Dibujado de cajas, textos, radar y smart zoom.
4. **Salida:** Video final `.mp4` con audio sincronizado.


## 6. Tecnologías Utilizadas

| Tecnología             | Uso principal                                      |
|------------------------|---------------------------------------------------|
| **Python 3.x**         | Lenguaje base                                     |
| **Ultralytics YOLOv8** | Detección de objetos                              |
| **ByteTrack**          | Tracking de objetos                               |
| **OpenCV**             | Procesamiento de imagen, CLAHE, contornos         |
| **EasyOCR**            | Lectura de texto en señales                       |
| **spaCy**              | Procesamiento de lenguaje natural (NLP)           |
| **Whisper**            | Transcripción de comandos de voz                  |
| **gTTS**               | Síntesis de voz (Text-to-Speech)                  |
| **MoviePy**            | Edición y montaje de video/audio                  |
| **NumPy**              | Operaciones numéricas                             |


## 7. Clases Detectadas (COCO)

| ID | Clase         | Uso en SafeStep                        |
|----|---------------|----------------------------------------|
| 0  | Person        | Detección de peatones                  |
| 1  | Bicycle       | Vehículo - Radar                       |
| 2  | Car           | Vehículo - Radar y alertas             |
| 3  | Motorcycle    | Vehículo - Radar                       |
| 5  | Bus           | Vehículo - Radar                       |
| 7  | Truck         | Vehículo - Radar                       |
| 9  | Traffic Light | Análisis de color (validación geométrica) |
| 11 | Stop Sign     | Trigger para OCR                       |


## 8. Instalación y Requisitos

```bash
pip install ultralytics opencv-python easyocr spacy gtts moviepy openai-whisper numpy
python -m spacy download es_core_news_sm
```

## 9. Justificaciones Técnicas y Decisiones de Diseño

Ver el apartado detallado en el notebook principal para conocer las decisiones clave sobre optimización, accesibilidad, gestión de memoria, audio y arquitectura modular.

## 10. Limitaciones y Posibles Mejoras

- La detección de movimiento puede verse afectada por el movimiento de cámara (falsos positivos).
- - El OCR puede confundir carteles con otros elementos visuales similares, aunque ahora se ejecuta sobre cualquier objeto identificado como cartel o señal, aumentando la cobertura pero también el riesgo de falsos positivos.
- En entornos urbanos muy estimulados, la sobrecarga de avisos puede ser un reto de usabilidad.
- Mejoras futuras: integración con mapas/GPS, modelos de deep learning específicos para pasos de cebra, personalización avanzada de avisos.

## 11. Agradecimientos y Referencias

- Proyecto desarrollado para la asignatura de Interacción Persona-Máquina - 3º Año.
- Basado en librerías y modelos open source: Ultralytics YOLOv8, OpenAI Whisper, spaCy, EasyOCR, MoviePy, OpenCV, gTTS, NumPy.