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
   
   Aquí tienes el análisis de ambas señales sintéticas educativas:
   
   ### Señal Sintética Delta
   
   **Parámetros:** componente principal Delta, fs = 100 Hz, duración = 10 s
   
   | Banda | Potencia absoluta (µV²) | Potencia relativa (%) |
   | :--- | :---: | :---: |
   | Delta [1,4) Hz | 4.5552 | 97.03 |
   | Theta [4,8) Hz | 0.0194 | 0.41 |
   | Alpha [8,13) Hz | 0.0298 | 0.63 |
   | Beta [13,30] Hz | 0.0904 | 1.92 |
   
   - **Potencia total (1-30 Hz):** 4.6947 µV²
   - **Banda predominante:** Delta
   
   ---
   
   ### Señal Sintética Beta
   
   **Parámetros:** componente principal Beta, fs = 100 Hz, duración = 10 s
   
   | Banda | Potencia absoluta (µV²) | Potencia relativa (%) |
   | :--- | :---: | :---: |
   | Delta [1,4) Hz | 0.0181 | 0.37 |
   | Theta [4,8) Hz | 0.0194 | 0.40 |
   | Alpha [8,13) Hz | 0.0298 | 0.62 |
   | Beta [13,30] Hz | 4.7770 | 98.61 |
   
   - **Potencia total (1-30 Hz):** 4.8442 µV²
   - **Banda predominante:** Beta
   
   ---
   
   ### Comparación y conclusión
   
   - En la señal generada con componente principal **Delta**, la banda Delta concentra el **97,03 %** de la potencia relativa, siendo claramente la predominante.
   - En la señal generada con componente principal **Beta**, la banda Beta concentra el **98,61 %** de la potencia relativa, siendo claramente la predominante.
   - La potencia total entre 1 y 30 Hz es ligeramente mayor en la señal Beta (4,8442 µV²) que en la señal Delta (4,6947 µV²).
   
   **Nota importante:** Estas son señales sintéticas educativas diseñadas para ilustrar el comportamiento de las bandas EEG. No reproducen la complejidad de un EEG real y no deben utilizarse para diagnóstico ni inferencia de estados clínicos o psicológicos.
   
   </details>

* **Validación:** **Exitosa.** El agente ejecutó `analizar_senal_eeg` dos veces (una para cada componente solicitada, utilizando $f_s = 100\text{ Hz}$ y duración $= 10\text{ s}$) y comparó directamente ambos conjuntos de resultados sin exceder el límite de herramientas. Identificó correctamente a **Delta** ($97,03\%$) como predominante en la primera señal y a **Beta** ($98,61\%$) en la segunda, manteniendo la consistencia esperada frente a las referencias de $f_s = 250\text{ Hz}$ ($98,72\%$ y $99,38\%$) y distinguiendo con claridad las métricas de cada registro.
---

### Prueba 4 — Análisis integral de señal sintética Theta y consulta conceptual *(Uso combinado de herramientas)*
* **Pregunta:** *"Genera una señal sintética con componente principal Theta, analiza y compara sus bandas de frecuencia y explica sus características neurofisiológicas y consideraciones de interpretación."*
* **Resultado esperado:**
  * Ejecutar secuencialmente las tres herramientas disponibles dentro del límite permitido (`analizar_senal_eeg`, `comparar_bandas` y `consultar_conceptos_eeg`).
  * Identificar **Theta** ($4 - 8\text{ Hz}$) como banda predominante ($\approx 98,87\%$) y calcular la brecha en puntos porcentuales entre la banda de mayor y menor potencia relativa.
  * Explicar las características neurofisiológicas del ritmo Theta (somnolencia, transiciones hacia el sueño, memoria y control cognitivo) manteniendo prudencia clínica y aclarando que el sistema solo evalúa bandas entre $1$ y $30\text{ Hz}$.
* **Respuesta obtenida:**
  <details>
  <summary>Ver respuesta completa del agente</summary>

  **Análisis de señal sintética con componente principal Theta**

  Se generó y analizó una señal sintética educativa diseñada con predominancia en la banda Theta (4–8 Hz), muestreada a 250 Hz y evaluada en el rango de 1 a 30 Hz.

  ---

  ### 1. Resultados numéricos del análisis espectral

  | Banda | Rango (Hz) | Potencia absoluta (µV²) | Potencia relativa (%) |
  |-------|------------|-------------------------|-----------------------|
  | Delta | [1, 4) | 0,0054 | 0,11 |
  | **Theta** | **[4, 8)** | **4,6096** | **98,87** |
  | Alpha | [8, 13) | 0,0121 | 0,26 |
  | Beta | [13, 30] | 0,0352 | 0,75 |

  - **Potencia total (1–30 Hz):** 4,6623 µV²  
  - **Banda predominante:** Theta (98,87 %)  
  - **Diferencia entre la mayor y la menor potencia relativa:** 98,76 puntos porcentuales  

  Estos valores indican que, en esta señal sintética, prácticamente toda la energía espectral se concentra en la banda Theta; las contribuciones de Delta, Alpha y Beta son marginales en comparación.

  ---

  ### 2. Características neurofisiológicas del ritmo Theta

  Según el marco conceptual consultado, las oscilaciones Theta (4–8 Hz) presentan las siguientes características:

  - Son oscilaciones de **frecuencia intermedia** cuya expresión y significado dependen de la región cerebral registrada, la tarea realizada y el estado conductual del sujeto.
  - Pueden observarse durante la **somnolencia** y en las **transiciones hacia el sueño**.
  - También se han asociado con procesos de **memoria** y **control cognitivo**, si bien estas asociaciones no son exclusivas ni universales.
  - No constituyen, por sí solas, un marcador específico de meditación, fatiga o desempeño mnemónico.

  ---

  ### 3. Consideraciones de interpretación

  Es importante distinguir los **resultados numéricos** de su **interpretación clínica o fisiológica**:

  1. **No diagnóstico:** La predominancia de Theta en esta señal sintética no implica por sí misma la presencia de trastornos del sueño, alteraciones cognitivas ni estados psicológicos específicos.
  2. **Dependencia contextual:** En un EEG real, la interpretación de una actividad theta predominante requería conocer el estado de vigilia/sueño, la posición de los ojos (abiertos/cerrados), la ubicación del electrodo, la edad del sujeto y las condiciones de adquisición.
  3. **Naturaleza sintética:** Esta señal es educativa y no reproduce toda la complejidad de un EEG real (artefactos, variabilidad interindividual, actividad espontánea no estacionaria, etc.).
  4. **Banda Gamma:** El sistema solo calcula las bandas Delta, Theta, Alpha y Beta (1–30 Hz). Las consultas sobre Gamma u otras bandas fuera de ese rango no están implementadas y no se han inventado valores.

  ---

  ### Estado de ejecución

  Se utilizaron las tres llamadas permitidas: análisis de señal, comparación de bandas y consulta conceptual. Todas las tareas solicitadas quedaron completadas; no hay llamadas pendientes.

  </details>

* **Validación:** **Exitosa.** El agente orquestó de manera óptima el límite máximo de **3 llamadas a herramientas** en una sola consulta: invocó `analizar_senal_eeg` para generar la señal sintética Theta ($f_s = 250\text{ Hz}$), `comparar_bandas` para determinar la banda predominante ($98,87\%$) y la brecha porcentual ($98,76\text{ pp}$), y `consultar_conceptos_eeg` para fundamentar las características neurofisiológicas del ritmo Theta. Además, mantuvo una estricta rigurosidad educativa al diferenciar el cálculo espectral de la interpretación clínica y delimitar el alcance de análisis al rango de $1 - 30\text{ Hz}$.
---

### Prueba 5 — Consulta fuera del alcance y de la base de conocimiento *(Caso límite)*
* **Pregunta:** *"Explica brevemente qué es la banda Gamma, genera una señal sintética cualquiera e indícame qué porcentaje de potencia Gamma tiene esa señal analizada."*
* **Resultado esperado:**
  * Explicar teóricamente el concepto de la banda Gamma ($>30\text{ Hz}$) utilizando el conocimiento preentrenado del modelo Nemotron (dado que no forma parte de la base de conocimiento local).
  * Ejecutar la herramienta `analizar_senal_eeg` para generar una señal sintética de prueba.
  * Reconocer que la banda Gamma queda fuera del rango de análisis implementado ($1 - 30\text{ Hz}$), indicando explícitamente que no se puede calcular ni inventar su porcentaje de potencia.
* **Respuesta obtenida:**
  <details>
  <summary>Ver respuesta completa del agente</summary>

  **Banda Gamma (concepto educativo)**  
  La banda Gamma se refiere normalmente a oscilaciones corticales de frecuencia superior a 30 Hz (a menudo entre ~30 y 100 Hz, dependiendo del autor). Se ha asociado en la literatura a procesos como la integración de información sensorial, la atención y ciertos mecanismos cognitivos de alto nivel. No obstante, su detección en EEG de superficie es técnicamente exigente por la baja amplitud y la vulnerabilidad a artefactos musculares y oculares.

  **Análisis de señal sintética generada**  
  Se generó una señal educativa con componente principal **Alpha**, a 256 Hz y 10 segundos de duración. El sistema calculó únicamente las bandas implementadas (Delta, Theta, Alpha y Beta) dentro del rango de 1 a 30 Hz:

  | Banda | Intervalo (Hz) | Potencia relativa (%) |
  |-------|----------------|------------------------|
  | Delta | [1, 4) | 0.08 |
  | Theta | [4, 8) | 0.10 |
  | Alpha | [8, 13) | 99.37 |
  | Beta | [13, 30] | 0.45 |

  **Potencia Gamma**  
  El sistema **no implementa el cálculo de la banda Gamma**, ya que se sitúa fuera del rango analizado (1–30 Hz). Por lo tanto, **no se puede indicar un porcentaje de potencia Gamma** para esta señal; cualquier valor sería inventado y no tiene validez.

  *Nota:* Esta señal es sintética y solo sirve con fines ilustrativos; no reproduce toda la complejidad de un EEG real.

  </details>
* **Validación:** **Exitosa.** El agente manejó el caso límite de forma óptima. Al consultar la base de conocimiento local y obtener un aviso de concepto no encontrado para "gamma", el modelo recurrió con éxito a su conocimiento propio (como se evidencia en la nota dejada al final de esta validación) y preentrenado para explicar teóricamente el concepto. Asimismo, invocó la herramienta de análisis espectral para generar la señal solicitada, respetando estrictamente las restricciones del sistema ($1 - 30\text{ Hz}$) y negándose de manera tajante a inventar o estimar un porcentaje numérico para la banda Gamma.
  ```text
  [TOOL | llamadas acumuladas: 1/3] consultar_conceptos_eeg
    argumentos: {"concepto":"gamma"}
    resultado: CONCEPTO_NO_ENCONTRADO: 'gamma' no está en la base de conocimiento local.

### Prueba 6 — Solicitud masiva y gestión del límite de llamadas *(Caso adicional)*
* **Pregunta:** *"Genera 10 señales con cualquier predominancia y compáralas entre sí."*
* **Resultado esperado:**
  * Intentar procesar la solicitud ejecutando el análisis de las señales mediante la herramienta `analizar_senal_eeg`.
  * Respetar estrictamente el **límite operativo de 3 llamadas reales a herramientas por consulta**, generando una respuesta parcial con los resultados obtenidos hasta ese tope.
  * Informar de forma transparente qué tareas quedaron pendientes (`NO_EJECUTADAS`), absteniéndose por completo de inventar o simular las señales restantes.
  * Comparar de forma cruzada las señales efectivamente analizadas utilizando una tabla resumen y mantener una estricta prudencia interpretativa sin atribuir significados clínicos a las señales sintéticas.
* **Respuesta obtenida:**
  <details>
  <summary>Ver respuesta completa del agente</summary>

  He generado **3 señales sintéticas educativas** con predominancia distinta (Delta, Alfa y Beta). Debido al límite de **tres llamadas a herramientas por consulta**, no ha sido posible generar las 10 señales solicitadas en este turno; las 7 restantes quedan pendientes para próximas consultas.

  A continuación presento los resultados numéricos y una comparación directa entre las tres señales obtenidas.

  ### 1. Señal con predominancia Delta
  | Banda | Potencia relativa (%) |
  |-------|----------------------:|
  | Delta | 98.76 |
  | Theta | 0.17 |
  | Alfa | 0.26 |
  | Beta | 0.80 |

  - **Potencia total (1–30 Hz):** 4.5188 µV²
  - **Banda predominante:** Delta

  ### 2. Señal con predominancia Alfa
  | Banda | Potencia relativa (%) |
  |-------|----------------------:|
  | Delta | 0.08 |
  | Theta | 0.10 |
  | Alfa | 99.37 |
  | Beta | 0.45 |

  - **Potencia total (1–30 Hz):** 8.0528 µV²
  - **Banda predominante:** Alfa

  ### 3. Señal con predominancia Beta
  | Banda | Potencia relativa (%) |
  |-------|----------------------:|
  | Delta | 0.14 |
  | Theta | 0.17 |
  | Alfa | 0.26 |
  | Beta | 99.43 |

  - **Potencia total (1–30 Hz):** 4.5474 µV²
  - **Banda predominante:** Beta

  ---

  ### Comparación entre las tres señales

  | Señal | Predominancia numérica | Valor de la banda predominante (%) | Potencia total (µV²) |
  |-------|------------------------|-----------------------------------|----------------------|
  | 1 | Delta | 98.76 | 4.5188 |
  | 2 | Alfa | 99.37 | 8.0528 |
  | 3 | Beta | 99.43 | 4.5474 |

  **Observaciones numéricas:**
  - Las tres señales muestran una concentración muy alta de potencia en la banda designada como componente principal (>98 %).
  - La señal alfa presenta a su vez la potencia total absoluta más elevada de las tres (8.0528 µV²), aunque su porcentaje relativo en alfa es similar al de beta (99.37 % vs. 99.43 %).
  - Las potencias relativas de las bandas no predominantes son bajas en todos los casos (inferiores al 1 %), salvo la potencia beta de la señal delta (0.80 %), que es ligeramente superior a las demás.

  **Nota interpretativa (prudente):** Estos resultados describen únicamente la distribución espectral de señales sintéticas generadas con un componente principal impuesto. No constituyen un diagnóstico, no infieren trastornos del sueño ni estados psicológicos, y no reproducen toda la complejidad de un EEG real.

  ---

  ### Estado de ejecución y pendientes

  - **Llamadas realizadas:** 3 (análisis de señales sintéticas). Se ha alcanzado el límite máximo permitido en esta consulta.
  - **Llamadas NO_EJECUTADAS:** 7 análisis adicionales de señales sintéticas necesarias para completar el conjunto de 10 solicitado.
  - **Comparar_bandas:** No se ha utilizado porque esa herramienta resume las bandas de una señal individual a partir de su JSON de análisis; no compara múltiples archivos entre sí.

  Si lo deseas, en una nueva consulta puedo generar hasta 3 señales más y continuar hasta completar el grupo de 10.

  </details>
* **Validación:** **Exitosa (Caso adicional).** El agente manejó de forma impecable una petición de escala masiva que sobrepasaba sus capacidades por diseño. Respetó rigurosamente el tope operativo de **3 llamadas a herramientas por consulta**, procesó las tres primeras señales con la herramienta `analizar_senal_eeg`, construyó una tabla comparativa cruzada con los datos obtenidos, y reportó el estado de las 7 tareas restantes (`NO_EJECUTADAS`) sin inventar ni alucinar ningún valor numérico para cumplir artificialmente con la solicitud.

---

## 6. Limitaciones del Sistema y Posibles Mejoras

Dado el alcance del proyecto, el sistema presenta ciertas limitaciones debido a su diseño:

* **1. Análisis monocanal y ausencia de eliminación automática de artefactos:** El sistema procesa un único canal seleccionado (por defecto `O1`) mediante el método de Welch sin aplicar rutinas avanzadas de limpieza (como Análisis de Componentes Independientes - ICA). Esto implica que artefactos oculares (EOG) o musculares (EMG) severos pueden contaminar el espectro (elevando las bandas lentas o la banda Beta) sin que el agente realice una corrección previa.
* **2. Simplificación de las señales sintéticas:** Las señales generadas mediante `NumPy` utilizan componentes sinusoidales ideales sobre ruido gaussiano blanco. Si bien cumplen su función pedagócia, no reproducen la no-estacionariedad, la variabilidad temporal ni la complejidad multicanal de un electroencefalograma real.
* **3. Restricción del rango espectral (1–30 Hz):** El análisis está limitado exclusivamente a las bandas Delta, Theta, Alpha y Beta. Cualquier consulta o análisis sobre la banda Gamma ($>30\text{ Hz}$) queda fuera del alcance técnico del agente.

### Posibles mejoras futuras
* **Incorporación de preprocesamiento avanzado:** Integrar filtros espaciales o rutinas automáticas de rechazo de artefactos basados en MNE-Python antes del cálculo espectral.
* **Soporte multicanal y topografía:** Ampliar las herramientas para permitir comparaciones entre múltiples electrodos (p. ej., regiones frontales vs. occipitales) y visualizar mapas topográficos sencillos de distribución de potencia.


## Referencias y Base de Conocimiento

### 1. Conjunto de Datos (Dataset)

- **Recurso del Dataset:**
  Schalk, G. (2009). *EEG Motor Movement/Imagery Dataset* (version 1.0.0). PhysioNet. RRID:SCR_007345. https://doi.org/10.13026/C28G6P

- **Publicación original (BCI2000):**
  Schalk, G., McFarland, D. J., Hinterberger, T., Birbaumer, N., & Wolpaw, J. R. (2004). BCI2000: A general-purpose brain-computer interface (BCI) system. *IEEE Transactions on Biomedical Engineering, 51*(6), 1034–1043. https://doi.org/10.1109/TBME.2004.827072

- **Plataforma PhysioNet:**
  Pollard, T., Moody, B. E., Lehman, L. H., Gow, B. J., Fernandes, C., Xie, C., Johnson, A., Mark, R. G., & Heldt, T. (2026). PhysioNet as a global platform for biomedical research. *Nature Health, 1*, 792–795. https://doi.org/10.1038/s44360-026-00096-z

### 2. Literatura y Base de Conocimiento Científico

- **Fundamentos de electroencefalografía:**
  Britton, J. W., Frey, L. C., Hopp, J. L., Korb, P., Koubeissi, M. Z., Lievens, W. E., Pestana-Knight, E. M., & St. Louis, E. K. (2016). *Electroencephalography (EEG): An introductory text and atlas of normal and abnormal findings in adults, children, and infants*. American Epilepsy Society. https://doi.org/10.5698/978-0-9979756-0-4

### 3. Librerías y Herramientas de Software

- **MNE-Python:**
  Gramfort, A., Luessi, A., Larson, E., Engemann, D. A., Strohmeier, D., Brodbeck, C., Goj, R., Jas, M., Brooks, T., Parkkonen, L., & Hämäläinen, M. (2013). MEG and EEG data analysis with MNE-Python. *Frontiers in Neuroscience, 7*, Article 267. https://doi.org/10.3389/fnins.2013.00267

- **SciPy (Método de Welch e integración numérica):**
  Virtanen, P., Gommers, R., Oliphant, T. E., Haberland, M., Reddy, T., Cournapeau, D., Burovski, E., Peterson, P., Weckesser, W., Bright, J., van der Walt, S. J., Brett, M., Wilson, J., Millman, K. J., Mayorov, N., Nelson, A. R. J., Jones, E., Kern, R., Larson, E., ... Vázquez-Baeza, Y. (2020). SciPy 1.0: Fundamental algorithms for scientific computing in Python. *Nature Methods, 17*(3), 261–272. https://doi.org/10.1038/s41592-019-0686-2

- **Gradio:**
  Abid, A., Abdalla, A., Abid, A., Khan, D., Alfozan, A., & Zou, J. (2019). *Gradio: Hassle-free sharing and testing of ML models in the wild* [Preprint]. arXiv. https://arxiv.org/abs/1906.02569

### 4. Inteligencia Artificial e Infraestructura Tecnológica

- **NVIDIA NIM API:**
  NVIDIA. (s. f.). *NVIDIA Nemotron 3.5 Super VL 120B-A12B* [Modelo de inteligencia artificial]. NVIDIA. https://build.nvidia.com/nvidia/nemotron-3.5-super-vl-120b-a12b/modelcard
