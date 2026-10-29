# Sistema Backend de Triaje y Resolución Inteligente con RAG (v4)

**Trabajo Práctico 4 – Seminario: Introducción a los LLMs y al desarrollo deaplicaciones asociadas - Generación Aumentada por Recuperación (RAG) para Soporte Automatizado**

---

## Descripción del Proyecto

Este proyecto constituye la **versión 2 (v2)** del motor de triaje y resolución inteligente de reclamos de E-Commerce desarrollado en el [TP3](https://github.com/Bertolini-Victor/TP3-Seminario-LLMs). En esta evolución arquitectónica, el sistema **elimina las políticas corporativas fijas (hardcodeadas)** del prompt de resolución (`Prompt 2.md`) e incorpora un **pipeline RAG (Retrieval-Augmented Generation)** completo sobre el manual oficial [`politicas_ecommerce_integral.md`](./politicas_ecommerce_integral.md).

Ante cada consulta o reclamo de un cliente:
1. Se recupera dinámicamente desde **ChromaDB** el conjunto Top-$K$ de cláusulas relevantes mediante similitud del coseno sobre embeddings densos multilingües (`sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2`).
2. Se inyecta el contexto recuperado en la variable de plantilla `{contexto_politicas}` delimitada con etiquetas XML `<contexto_politicas>`.
3. Se ejecuta el pipeline en dos etapas (Triaje Few-Shot en `Prompt 1.md` $\rightarrow$ Resolución con *Chain-of-Thought* guiado en `Prompt 2.md`).
4. Se valida estrictamente la salida JSON mediante un esquema **Pydantic (`RespuestaSoporteRAG`)** y el validador nativo del TP3 (`validar_esquema_salida`), garantizando trazabilidad textual en `politica_aplicada` y resistencia frente a políticas inventadas (*grounding*) o ataques de *Prompt Injection*.

---

## Estructura del Repositorio

* `TP4_Resolucion.ipynb`: **Entregable principal**. Jupyter Notebook ejecutable en español con las 7 secciones requeridas, tablas empíricas, gráficos/heatmaps y todas las salidas visibles.
* `politicas_ecommerce_integral.md`: Corpus oficial de políticas de envíos, garantías, facturación/notas de crédito y botón de arrepentimiento.
* `Prompt 1.md`: Plantilla de la Etapa 1 (Triaje y Clasificación con 10 ejemplos *few-shot* y aislamiento `<reports>`).
* `Prompt 2_TP3.md`: Plantilla original de la Etapa 2 del TP3 (conservada para comparación y cálculo de `diff` antes/después).
* `Prompt 2.md`: Plantilla evolucionada de la Etapa 2 para el TP4 (sin políticas hardcodeadas, con inyección de `<contexto_politicas>{contexto_politicas}</contexto_politicas>` y reglas estrictas de *grounding*).
* `TP4.md`: Enunciado oficial y criterios de evaluación del Trabajo Práctico 4.
* `requirements.txt`: Dependencias reproducibles del entorno.
* `.env.example`: Plantilla para configuración de variables de entorno (`OPENAI_API_KEY`, `OPENAI_BASE_URL`, `MODEL_NAME`).

---

## Estructura del Notebook (`TP4_Resolucion.ipynb`)

1. **Configuración del entorno y librerías:** Instalación reproducible, semillas fijas (`SEED = 42`), inicialización del tokenizador `tiktoken` (`cl100k_base`), modelo de embeddings, `ChromaDB`, esquemas `Pydantic` y cliente `OpenAI`. Justificación técnica del stack local ligero.
2. **Pipeline de indexado:** Carga y análisis estadístico del corpus; implementación de `indexar(chunk_size, overlap_pct)` midiendo en tokens para las 9 combinaciones (tamaños de chunk: 10, 100 y 1000 tokens × solapamientos: 0%, 10% y 80%), análisis de casos degenerados y comparación con fragmentación estructural por encabezados Markdown (`MarkdownHeaderTextSplitter`).
3. **Evaluación de las 9 combinaciones (Calidad y Costo):** Mini-benchmark verificable de 10 consultas con *ground truth* por sección y evidencias normativas; cálculo de `Hit Rate@k`, `Keyword Recall@k`, `MRR`, cantidad de chunks, tokens indexados, tokens inyectados por consulta y estimación en USD; tabla $3 \times 3$ y heatmaps comparativos.
4. **Mecanismo de recuperación (`recuperar_contexto`):** Implementación con similitud coseno y metadatos de sección; evaluación empírica de $k \in \{2, 3, 4\}$ en calidad y costo.
5. **Pipeline RAG completo (`procesar_mensaje_con_rag`):** Comparativa `diff` entre `Prompt 2_TP3.md` y `Prompt 2.md`, validación estricta con Pydantic (`RespuestaSoporteRAG`), manejo de reintentos y defensas anti-*Prompt Injection*.
6. **Evaluación de casos de negocio (Casos Borde):** Evaluación detallada de 4 casos críticos (cafetera rota a los 45 días de la entrega vs. arrepentimiento vencido, nota de crédito fuera del plazo de 15 días hábiles desde emisión de factura, cliente que inventa una política falsa de 60 días / 1 año de garantía, y consulta fuera del corpus).
7. **Documentación y justificación técnica:** Justificación cuantitativa del *chunking* y de $k$, **experimento controlado de inyección de ruido** en el contexto recuperado, limitaciones del sistema y procedimiento de actualización del manual sin modificar el prompt.

---

## Instrucciones de Ejecución

### 1. Crear y activar el entorno virtual
```bash
python -m venv venv
# En Windows (PowerShell):
.\venv\Scripts\Activate.ps1
# En Linux / macOS:
source venv/bin/activate
```

### 2. Instalar dependencias
```bash
pip install -r requirements.txt
```

### 3. Configurar variables de entorno
```bash
cp .env.example .env
```
Podés configurar tu `OPENAI_API_KEY` (OpenAI, Groq, Google Gemini compatible con OpenAI o servidor local Ollama). Si no se configura una clave externa, el notebook incluye el motor determinista de respaldo consciente del contexto RAG (`simular_respuesta_llm_rag`) que analiza dinámicamente el bloque `<contexto_politicas>` recuperado de ChromaDB para garantizar reproducibilidad offline de punta a punta.

### 4. Ejecutar el Notebook
```bash
jupyter notebook TP4_Resolucion.ipynb
```
O para ejecutar todas las celdas desde consola:
```bash
jupyter nbconvert --to notebook --execute --inplace TP4_Resolucion.ipynb
```
