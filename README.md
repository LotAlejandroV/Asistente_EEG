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

## 5. Casos de Prueba y Resultados

### Prueba 1 — Análisis de una señal sintética Alpha *(Habitual 1)*
* **Pregunta:** *"Genera una señal sintética con componente principal Alpha. Analiza sus potencias absolutas y relativas e identifica la banda predominante."*
* **Resultado esperado:**
  * Ejecutar `analizar_senal_eeg` e identificar **Alpha** como banda predominante.
  * Las potencias relativas deben sumar aproximadamente $100\%$, coincidiendo con los valores de referencia:
    | Banda | Potencia relativa de referencia |
    | :--- | :---: |
    | Delta | $0,07\%$ |
    | Theta | $0,14\%$ |
    | Alpha | $99,36\%$ |
    | Beta | $0,43\%$ |
    
* **Respuesta obtenida:**
   <details>
      <summary>Ver respuesta completa del agente</summary>
   
   A continuación presento los resultados del análisis de la señal sintética generada con componente principal **Alpha**.
   
   ### Parámetros de la señal
   - **Tipo:** Señal sintética educativa (componente principal: alpha)
   - **Frecuencia de muestreo:** 256 Hz
   - **Duración:** 10 s
   - **Canal analizado:** Sintético
   - **Rango analizado:** 1 – 30 Hz
   
   ### Potencias absolutas (µV²)
   
   | Banda | Rango (Hz) | Potencia absoluta (µV²) |
   | :--- | :---: | :---: |
   | Delta | [1, 4) | 0,0064 |
   | Theta | [4, 8) | 0,0079 |
   | Alpha | [8, 13) | 8,0021 |
   | Beta | [13, 30] | 0,0364 |
   
   ### Potencias relativas (respecto al total 1–30 Hz)
   
   La potencia total entre 1 y 30 Hz fue de **8,0528 µV²**.
   
   | Banda | Potencia relativa |
   | :--- | :---: |
   | Delta | 0,08 % |
   | Theta | 0,10 % |
   | Alpha | **99,37 %** |
   | Beta | 0,45 % |
   
   ### Banda predominante
   La banda predominante es **Alpha**, concentrando casi la totalidad de la potencia de la señal en el rango analizado.
   
   ---
   
   **Nota importante:** Esta señal es sintética y tiene fines educativos. No reproduce toda la complejidad de un EEG real, por lo que estos resultados no deben utilizarse para inferir estados fisiológicos, psicológicos ni para diagnosticar ninguna condición.
   
   </details>

* * **Validación:** **Exitosa.** El agente seleccionó correctamente la herramienta `analizar_senal_eeg` para generar la señal sintética con componente Alpha (configurando parámetros válidos de $f_s = 256\text{ Hz}$ y duración de $10\text{ s}$). Identificó con precisión a **Alpha** como la banda predominante ($99,37\%$), y las potencias relativas reportadas suman exactamente el $100,00\%$, manteniendo plena consistencia con los valores de referencia ($99,36\%$). Asimismo, aclaró de forma explícita el carácter educativo de la señal sintética sin realizar afirmaciones diagnósticas.

---

### Prueba 2 — Consulta conceptual sobre Alpha *(Habitual 2)*
* **Pregunta:** *"¿Qué ocurre con la actividad Alpha cuando una persona abre los ojos? Explica el fenómeno y su interpretación fisiológica."*
* **Resultado esperado:**
  * Consultar `consultar_conceptos_eeg`.
  * Explicar que la actividad Alpha posterior suele atenuarse al abrir los ojos o durante tareas de procesamiento visual (*reactividad Alpha*).
  * Mantener prudencia interpretativa sin afirmar que Alpha mide directamente calma, bienestar o un estado psicológico.
* **Respuesta obtenida:**

   <details>
   <summary>Ver respuesta completa del agente</summary>
   
   Cuando una persona abre los ojos, la actividad alfa posterior (aproximadamente 8–13 Hz) suele **atenuarse o disminuir en potencia**, fenómeno denominado **bloqueo alfa** o **reactividad alfa**.
   
   ### Qué ocurre
   En adultos despiertos y relajados con los ojos cerrados, el ritmo alfa es especialmente visible en regiones posteriores del cuero cabelludo (zona occipital/parietal). Al abrir los ojos, la entrada visual y el procesamiento de estímulos externos provocan una reducción de esa actividad alfa posterior. No se trata de que la actividad “desaparezca” por completo en todos los casos, pero sí de una supresión o desplazamiento relativo de la potencia en esa banda.
   
   ### Interpretación fisiológica
   Desde un punto de vista fisiológico, este cambio se interpreta como un ajuste cortical a la nueva demanda sensorial:
   
   1. **Cambio de estado de vigilia**: con los ojos cerrados y poca estimulación externa, predomina un estado de reposo con atención dirigida hacia dentro. Al abrir los ojos, el cerebro pasa a un modo de procesamiento de información visual y vigilancia del entorno.
   2. **Activación de la corteza visual**: la estimulación retiniana y la activación de áreas visuales corticales interfieren con la generación o sostenimiento del ritmo alfa occipital.
   3. **Mecanismos de atención**: la apertura de los ojos está asociada a un aumento de la alerta y a un desplazamiento del foco atencional hacia el exterior, lo que modula las oscilaciones corticales en la banda alfa.
   
   ### Cautelas interpretativas
   - La disminución de alfa al abrir los ojos es un **fenómeno fisiológico normal y esperable**; por sí sola no permite diagnosticar enfermedades ni inferir estados emocionales, trastornos del sueño ni condiciones psicológicas.
   - Existe variabilidad entre personas; la magnitud de la atenuación puede depender de factores como la edad, el nivel de relajación previa, la iluminación y la tarea visual realizada.
   - Una mayor o menor potencia relativa alfa no demuestra por sí sola calma, bienestar ni ningún estado psicológico determinado.
   
   En resumen: abrir los ojos suele producir una **reducción de la actividad alfa posterior** como reflejo de la activación visual y del cambio de atención desde un estado interno-relajado hacia el procesamiento de estímulos externos.
   
   </details>

* **Validación:** **Exitosa.** El agente consultó la herramienta `consultar_conceptos_eeg` para recuperar los fundamentos neurofisiológicos de la banda Alpha desde la base de conocimiento local. Explicó con claridad el fenómeno de **reactividad o bloqueo alfa** (atenuación posterior entre $8$ y $13\text{ Hz}$ ante la estimulación visual y el cambio de foco atencional) y mantuvo una estricta prudencia interpretativa, aclarando que la potencia Alpha no mide por sí sola calma, bienestar ni estados psicológicos o clínicos.

---

### Prueba 3 — Comparación de dos señales sintéticas *(Habitual 3)*
* **Pregunta:** *"Genera una señal sintética Delta y otra Beta. Analiza ambas, compara sus potencias relativas e indica cuál es la banda predominante en cada una."*
* **Resultado esperado:**
  * Ejecutar `analizar_senal_eeg` para cada señal y distinguir correctamente ambas distribuciones, mostrando dos visualizaciones en paralelo.
  * Identificar predominancia **Delta** en la primera señal y **Beta** en la segunda, coincidiendo con las referencias numéricas:
    | Banda | Señal Delta (ref.) | Señal Beta (ref.) |
    | :--- | :---: | :---: |
    | Delta | $98,72\%$ | $0,11\%$ |
    | Theta | $0,25\%$ | $0,25\%$ |
    | Alpha | $0,26\%$ | $0,26\%$ |
    | Beta | $0,76\%$ | $99,38\%$ |
* **Respuesta obtenida:**
  <details>
  <summary>Ver respuesta completa del agente</summary>

  > *(Pega aquí la respuesta textual generada por el agente)*

  </details>
* **Validación:** *(Escribe aquí tu valoración breve sobre la comparación entre señales y la generación de las dos gráficas)*

---

### Prueba 4 — Consulta sobre artefactos EOG y EMG *(Uso de herramienta)*
* **Pregunta:** *"¿Qué artefactos EOG y EMG pueden contaminar una señal EEG? Explica cómo pueden afectar las frecuencias bajas y la banda Beta."*
* **Resultado esperado:**
  * Invocar `consultar_conceptos_eeg` para recuperar la información de artefactos de la base de conocimiento local.
  * Explicar que los artefactos oculares (**EOG**: parpadeos y movimientos oculares) aportan deflexiones de gran amplitud en frecuencias bajas, mientras que los musculares (**EMG**) contaminan frecuencias altas, incluida la banda **Beta**.
  * Advertir que estas contribuciones no cerebrales pueden alterar la interpretación espectral.
* **Respuesta obtenida:**
  <details>
  <summary>Ver respuesta completa del agente</summary>

  > *(Pega aquí la respuesta textual generada por el agente)*

  </details>
* **Validación:** *(Escribe aquí tu valoración breve sobre la información recuperada y la claridad pedagógica)*

---

### Prueba 5 — Consulta fuera del alcance de la base de conocimiento *(Caso límite)*
* **Pregunta:** *"¿Qué es la banda Gamma y qué porcentaje de potencia Gamma tiene la señal que analizaste?"*
* **Resultado esperado:**
  * Reconocer que la banda Gamma ($>30\text{ Hz}$) no está incluida dentro de las cuatro bandas implementadas ni en la base de conocimiento local.
  * Aclarar que el análisis espectral del sistema se limita al intervalo de $1 - 30\text{ Hz}$.
  * Abstenerse por completo de inventar o estimar un porcentaje de potencia para Gamma.
* **Respuesta obtenida:**
  <details>
  <summary>Ver respuesta completa del agente</summary>

  > *(Pega aquí la respuesta textual generada por el agente)*

  </details>
* **Validación:** *(Escribe aquí tu valoración breve sobre cómo el agente manejó el límite de dominio sin alucinar)*
