# TP2 - Procesamiento del Lenguaje Natural: Chatbot de Parks

## Descripción del Proyecto (Resumen del Trabajo Práctico)

Este Trabajo Práctico (TP) se centra en el desarrollo de un chatbot avanzado para el juego de mesa **Parks**, utilizando diversas técnicas de Procesamiento del Lenguaje Natural (PLN) y Arquitecturas de Recuperación Aumentada por Generación (RAG). El objetivo principal es construir un sistema capaz de responder de manera precisa y contextualizada a preguntas de los usuarios sobre las reglas, mecánicas, estadísticas y el lore del juego.

El proyecto aborda el desafío de integrar múltiples fuentes de información y mecanismos de búsqueda:

1.  **Carga y Preprocesamiento de Datos:** Se gestiona la ingesta de datos no estructurados (transcripciones de videos, reseñas, guías), datos tabulares (estadísticas del juego) y datos relacionales (un grafo de conocimiento) específicos de Parks. Esto incluye la detección de duplicados y la preparación para la fragmentación.

2.  **Text Splitting Inteligente:** Se implementan estrategias de fragmentación de texto adaptadas al tipo de contenido: una fragmentación por tokens para textos no estructurados y una fragmentación semántica avanzada (usando embeddings del modelo "Qwen/Qwen3-Embedding-0.6B") para documentos estructurados, asegurando la coherencia contextual de los "chunks".

3.  **Bases de Datos Híbridas para Recuperación:** Se configuran y utilizan distintas bases de datos para optimizar la recuperación de información:
    *   **Base de Datos Vectorial (ChromaDB):** Para búsquedas de similitud semántica.
    *   **Búsqueda BM25:** Para búsquedas basadas en la relevancia de palabras clave.
    *   **Base de Datos Tabular (Pandas):** Para consultas de datos cuantitativos.
    *   **Base de Grafos (RedisGraph):** Para explorar relaciones entre entidades del juego.

4.  **Optimización de Resultados de Búsqueda:** Se integra un modelo de Re-ranking ("BAAI/bge-reranker-v2-m3") para mejorar la pertinencia de los documentos recuperados por las búsquedas híbridas (combinación de BM25 y vectorial).

5.  **Clasificación de Intención y Routing:** Un Modelo de Lenguaje Grande (LLM) es empleado para clasificar la intención de la consulta del usuario, decidiendo dinámicamente qué tipo de base de datos es la más adecuada para responder (documentos, tablas, grafos o fuera de contexto), lo que permite un enrutamiento inteligente de la consulta.

6.  **Generación de Respuestas con Agente ReAct:** El núcleo conversacional se basa en un LLM (gemini-2.5-flash-preview-05-20) y un agente que implementa el patrón ReAct. Este agente razona sobre la pregunta, selecciona y ejecuta herramientas de búsqueda (incluyendo bases de datos internas y fuentes externas como Wikipedia o DuckDuckGo si es necesario) y, finalmente, sintetiza una respuesta coherente y relevante para el usuario. El enfoque ReAct garantiza que el agente recoja toda la información necesaria antes de formular una respuesta final, siguiendo un ciclo `Thought -> Action -> Observation`.

El resultado es un chatbot robusto y versátil que demuestra la integración de múltiples componentes de PLN y IA para crear una experiencia de usuario informada y precisa.

## Funcionalidades Clave

### 1. Carga de Datos y Preprocesamiento

-   **Lectura de Archivos:** Carga automática de archivos de texto desde el directorio `datos/informacion`.
-   **Detección de Duplicados:** Evita procesar contenido textual repetido.
-   **Detección de Idioma:** Identifica el idioma de cada documento.

### 2. Text Splitting

-   **Fragmentación por Tokens:** Para transcripciones de videos (texto no estructurado) con un tamaño de chunk de 512 tokens.
-   **Fragmentación Semántica:** Para documentos estructurados, utilizando el modelo de embedding "Qwen/Qwen3-Embedding-0.6B" para agrupar oraciones con similitud semántica (chunks de 2 a 1024 tokens).
-   **Filtrado de Chunks:** Eliminación de chunks muy cortos (menos de 500 caracteres) que se consideran de bajo/ningún valor.

### 3. Bases de Datos

-   **Base de Datos Vectorial (ChromaDB):** Almacena los embeddings de los chunks de texto para búsquedas semánticas. Utiliza el mismo modelo "Qwen/Qwen3-Embedding-0.6B".
-   **Búsqueda BM25:** Implementación del algoritmo BM25 para búsquedas basadas en la frecuencia de términos.
-   **Base de Datos Tabular (Pandas DataFrame):** Carga y preprocesa los datos estadísticos del juego, pivotando el DataFrame para un acceso más directo a la información.
-   **Base de Grafos (RedisGraph):** Construye un grafo de conocimiento a partir de relaciones predefinidas, permitiendo consultas sobre entidades y sus conexiones.

### 4. Componentes de Búsqueda

-   **Reranker (BAAI/bge-reranker-v2-m3):** Reordena los resultados de búsqueda combinados de BM25 y vectorial para mejorar la relevancia.
-   **Hybrid Search:** Combina los resultados de la búsqueda vectorial y BM25, eliminando duplicados y aplicando el reranking.
-   **Clasificador de Intención (LLM - Few-Shot):** Utiliza un LLM para clasificar la intención de la consulta del usuario, dirigiendo la búsqueda a la base de datos más adecuada (documental, tabular, grafos o "None" si no es relevante).
-   **Retrieval:** Orquesta el proceso de recuperación, seleccionando la fuente de datos según la intención clasificada.

### 5. Chatbot Conversacional

-   **LLM (gemini-2.5-flash-preview-05-20):** El cerebro del chatbot, utilizado para generar respuestas coherentes basadas en el contexto recuperado.
-   **Historial de Conversación:** Mantiene el contexto del diálogo para respuestas más fluidas.
-   **Agente ReAct:** Un agente que razona sobre la consulta, planifica el uso de las herramientas de búsqueda (internas y externas como Wikipedia/DuckDuckGo) en un ciclo `Thought -> Action -> Observation`, y sintetiza la respuesta final.
