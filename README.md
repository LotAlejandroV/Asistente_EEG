# Asistente-EEG: Asistente Educativo para el Análisis Espectral de Señales EEG

**Asignatura:** SI3003 — Inteligencia Artificial  
**Estudiante:** Alejandro Lotero Valencia  
**Alternativa seleccionada:** Opción B — Agente con herramientas  

---

## 1. Definición del Problema

El electroencefalograma (EEG) registra la actividad eléctrica cerebral mediante oscilaciones agrupadas en bandas de frecuencia (**Delta, Theta, Alpha y Beta**). Para estudiantes de áreas biomédicas o afines, comprender la relación entre la señal temporal, su espectro de frecuencias (Densidad Espectral de Potencia), el cálculo de potencias relativas y su interpretación neurofisiológica suele resultar complejo.

* **¿Qué resuelve y para quién?** Ofrece a estudiantes un entorno interactivo en lenguaje natural para explorar señales EEG (sintéticas o reales en formato `.edf`), visualizar sus espectros y consultar conceptos neurofisiológicos fundamentados.
* **¿Por qué un agente con herramientas?** El problema exige combinar cálculos numéricos deterministas (transformada espectral mediante el método de Welch e integración trapezoidal) con consultas a una base de conocimiento estructurada, seleccionando dinámicamente la función adecuada según la intención del usuario.
* **Alcance y limitaciones:** El sistema tiene fines **estrictamente educativos**. **No realiza diagnósticos clínicos**, no infiere patologías ni estados psicológicos individuales, y limita su análisis espectral al intervalo de $1$ a $30\text{ Hz}$ (bandas Delta, Theta, Alpha y Beta).

---

## 2. Arquitectura y Herramientas del Agente

El agente utiliza el modelo `nvidia/nemotron-3.5-super-vl-120b-a12b` a través de una interfaz compatible con OpenAI (`NVIDIA NIM`). Implementa un bucle iterativo de *function calling* adaptado para mantener el historial conversacional en **Gradio**, extraer metadatos de visualización sin saturar el contexto del LLM y aplicar un **límite estricto de hasta 3 llamadas reales a herramientas por consulta** (generando respuestas parciales sin inventar datos si se alcanza el tope).

### Herramientas implementadas
1. **`analizar_senal_eeg(ruta_archivo, canal_objetivo, fs, duracion, componente_principal)`**: Carga un archivo `.edf` mediante `MNE` (aplicando filtro pasa-altos a $1\text{ Hz}$) o genera una señal sintética educativa. Estima la Densidad Espectral de Potencia ($\text{PSD}$) mediante el método de **Welch** (ventana de Hann, $50\%$ de solapamiento) y calcula las potencias absolutas ($\mu\text{V}^2$) y relativas ($\%$) entre $1$ y $30\text{ Hz}$.
2. **`consultar_conceptos_eeg(concepto)`**: Consulta una base de conocimiento local estructurada en JSON (`knowledge_base/conceptos_eeg.json`), basada en literatura de la *American Epilepsy Society*, sobre características neurofisiológicas de las bandas, artefactos frecuentes (EOG, EMG, red eléctrica) y limitaciones interpretativas.
3. **`comparar_bandas(datos_json)`**: Valida que las potencias relativas sumen $\approx 100\%$, identifica la banda predominante y calcula la brecha en puntos porcentuales entre la banda de mayor y menor potencia.

---

## 3. Fuentes de Datos y Recursos

* **Base de conocimiento local (`knowledge_base/conceptos_eeg.json`):** Compendio educativo generado automáticamente por el cuaderno a partir de *Electroencephalography: An Introductory Text and Atlas* (*American Epilepsy Society*).
* **Señales sintéticas:** Generadas internamente con `NumPy` ($f_s = 250\text{ Hz}$, ruido gaussiano base + componente sinusoidal dominante configurable en Delta $2\text{ Hz}$, Theta $6\text{ Hz}$, Alpha $10\text{ Hz}$ o Beta $20\text{ Hz}$).
* **Registros reales (`.edf`):** Archivos de ejemplo alojados en la carpeta `data/` del repositorio (compatibles con estándar europeo EDF y procesados mediante la librería `MNE-Python`).

---

## 4. Instrucciones de Ejecución

El proyecto no incluye claves de API en el código ni en el repositorio. Para ejecutarlo, sigue estos pasos según tu entorno:

### En Google Colab
1. **Obtén tu clave de API:** Ingresa a [NVIDIA NIM / Build](https://build.nvidia.com), inicia sesión y genera tu credencial gratuita.
2. **Configura el secreto:** Abre `Asistente_EEG.ipynb` en Google Colab, haz clic en el ícono de **Secretos** (la llave 🔑 en el panel izquierdo), crea un nuevo secreto llamado `NVIDIA_API_KEY`, pega tu clave y activa el interruptor de acceso al cuaderno.
3. **Ejecuta el cuaderno:** Selecciona **Entorno de ejecución > Ejecutar todas**. La primera celda instalará las dependencias automáticamente y al final se desplegará el enlace público de la interfaz web en Gradio.

### En entorno local
1. **Obtén tu clave de API:** Genera tu credencial gratuita en [NVIDIA NIM / Build](https://build.nvidia.com).
2. **Configura la variable de entorno:** Crea un archivo `.env` en la raíz del proyecto (ignorado por Git) y añade la siguiente línea:
   ```env
   NVIDIA_API_KEY="tu_clave_aqui"
