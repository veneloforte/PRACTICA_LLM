# Informe Final - Asistente Turistico de Tenerife

## 1. Diseno de la solucion

### 1.1 Objetivo
Desarrollar un asistente conversacional especializado en turismo de Tenerife capaz de responder con contexto documental, citar fuentes y resolver ciertas consultas con una herramienta externa simulada (meteorologia).

### 1.2 Arquitectura general
La solucion sigue un enfoque RAG (Retrieval-Augmented Generation) con LangChain:

1. **Ingestion del conocimiento**
- Fuente: PDF turistico de Tenerife.
- Carga de documento con `PyPDFLoader`.
- Division en fragmentos con `RecursiveCharacterTextSplitter`.

2. **Representacion semantica**
- Embeddings con `GoogleGenerativeAIEmbeddings` usando `models/gemini-embedding-001`.

3. **Indexacion y recuperacion**
- Almacenamiento de vectores en FAISS en memoria.
- Recuperacion top-k con retriever para inyectar contexto relevante en el prompt.

4. **Generacion de respuestas**
- Modelo de chat de Gemini integrado con `ChatGoogleGenerativeAI` en LangChain.
- El prototipo arranco sobre familia Gemini 1.5 Flash y, por compatibilidad de disponibilidad de modelos en la API, se utilizo `gemini-flash-latest` durante la validacion final.

5. **Function Calling**
- Herramienta `get_weather(fecha)` para simulacion de prediccion meteorologica.
- Flujo de doble vuelta: el LLM decide llamar herramienta, se ejecuta en Python y se devuelve resultado al modelo para redactar respuesta final.

### 1.3 Control de calidad de respuesta
- Prompt de sistema orientado a dominio turistico de Tenerife.
- Inyeccion de metadata de pagina en el contexto recuperado para favorecer trazabilidad y citacion.
- Truncamiento de historial para contener costo de tokens.

## 2. Decisiones tecnicas

### 2.1 Tamano de chunk y solapamiento
Se eligio `chunk_size=1000` y `chunk_overlap=200` por equilibrio entre:

- **Cobertura semantica**: 1000 caracteres conservan suficiente contexto por fragmento para responder preguntas de turismo (lugares, descripciones, recomendaciones).
- **Continuidad contextual**: 200 de solapamiento reducen el riesgo de cortar conceptos en bordes de chunk.
- **Costo/rendimiento**: chunks mas pequenos mejoran granularidad pero aumentan numero de embeddings y latencia total; chunks demasiado grandes degradan precision de recuperacion.

Este valor mostro buen compromiso practico en las pruebas del notebook.

### 2.2 Embeddings de Google y recuperacion
El uso de embeddings de Google permite alinear el espacio semantico de recuperacion con el ecosistema Gemini. La recuperacion `top-k` se configuro con `k=3` para limitar ruido y mantener contexto breve pero relevante.

### 2.3 Inyeccion de metadatos para citas
Cada fragmento recuperado se transforma con una marca de fuente por pagina (por ejemplo, `--- Fuente: Pagina X ---`) antes de construir el prompt. Esta estrategia facilita:

- Trazabilidad de respuestas.
- Citacion explicita en salida final.
- Auditoria manual de veracidad sobre el documento.

## 3. Resultados

### 3.1 Observaciones de la celda de evaluacion
En la evaluacion de cuatro prompts se observaron las siguientes latencias:

- Q1 (RAG simple): **2.51 s**
- Q2 (RAG complejo): **4.63 s**
- Q3 (Function Calling): **4.74 s**
- Q4 (fuera de contexto): **13.21 s**

### 3.2 Lectura del grafico
- Las consultas RAG directas presentan latencias moderadas.
- La llamada a herramienta incrementa latencia por la segunda invocacion del modelo (pipeline tool-call + respuesta final).
- La consulta fuera de dominio fue la mas lenta, consistente con mayor esfuerzo de razonamiento/filtrado contextual.

### 3.3 Comportamiento cualitativo
Ante la pregunta fuera de contexto (receta de paella), el asistente indico que el documento no contenia esa informacion, mostrando comportamiento conservador y menor tendencia a alucinar.

## 4. Limitaciones y mejora futura

### 4.1 Limitaciones actuales
- **Memoria conversacional**: truncamiento estricto del historial a un numero fijo de mensajes.
- **FAISS en memoria local**: no hay persistencia ni escalado distribuido por defecto.
- **Tooling meteorologico simulado**: la herramienta de clima no consume API real.
- **Dependencia de disponibilidad de modelos**: algunos IDs de modelo cambian o dejan de estar disponibles segun la cuenta/API version.

### 4.2 Mejoras propuestas
- Sustituir FAISS local por base vectorial gestionada (por ejemplo, Pinecone o equivalente cloud) para escalabilidad, persistencia y observabilidad.
- Implementar memoria dinamica basada en resumen (p. ej., enfoque tipo `ConversationSummaryMemory`) para preservar contexto largo con menor costo de tokens.
- Integrar validacion de fechas robusta (NLP + normalizacion ISO 8601) en `get_weather`.
- Separar pipeline en modulos Python fuera del notebook para facilitar pruebas unitarias, CI y mantenibilidad.
- Incorporar evaluacion automatica de factualidad y citas.

## 5. Conclusiones
El prototipo cumple el objetivo de asistente turistico RAG para Tenerife, con arquitectura clara y resultados funcionales. Se validaron respuestas contextualizadas, citacion de fuentes y uso de herramienta externa. Las mejoras propuestas apuntan a robustez en produccion, escalabilidad y mejor gestion de contexto conversacional.
