Sistema RAG para consultas bibliográficas 📚🤖
Tabla de contenidos
Descripción
Problema que resuelve
Resumen del pipeline RAG
Arquitectura del sistema
Detalles técnicos
Stack tecnológico
Corpus de documentos
Instalación y uso local
Ejemplos de consultas
Decisiones de diseño
Limitaciones conocidas
Mejoras futuras
Troubleshooting
Estructura del proyecto
Autor
Descripción
Un sistema RAG (Recuperación Aumentada por Generación) diseñado para docentes de IPC que necesiten:

chequear información en la fuente,
parafrasear ideas,
buscar información específica,
comparar autores.
Corpus principal: "Desenredando la ciencia" (Buacar, Eudeba, 2022).
Utiliza búsqueda semántica con ChromaDB y generación con Gemini. Desarrollado como Trabajo Integrador N°2 para la materia Técnicas de Procesamiento del Habla (IFTS 24).

Problema que resuelve ❓
Los docentes requieren un mecanismo fiable para verificar y extraer información específica de textos base (por ejemplo, para formular preguntas de examen) sin depender de respuestas generadas fuera de la fuente.
La arquitectura RAG limita las respuestas del LLM a fragmentos recuperados de la base de datos vectorial, reduciendo alucinaciones y mejorando la trazabilidad.

Resumen del pipeline RAG 🔁
Usuario formula una pregunta.
Recuperador local (ChromaDB) obtiene los k=3 fragmentos más relevantes.
Se genera un prompt que combina la pregunta y los fragmentos.
Gemini genera la respuesta basada en esos fragmentos.
Se devuelve la respuesta junto con los documentos fuente.
Parámetros clave:

chunk_size = 500 caracteres
chunk_overlap = 50 caracteres
embeddings: intfloat/multilingual-e5-large (local)
vectorstore: ChromaDB (persist_directory="./chroma_db")
retrieval: top-k = 3
generación: models/gemini-2.5-flash (temperature=0.1)
Arquitectura del sistema 🧭
Mermaid
graph TD
    A[Usuario: Pregunta] --> B[Sistema RAG: RetrievalQA]
    B --> C{Retriever: Búsqueda de Embeddings}
    C --> D[Vector Store: ChromaDB]
    D --> E[Embeddings: intfloat/multilingual-e5-large]
    E --> C
    C --> F[Fragmentos de Documentos Relevantes]
    F --> G{LLM: ChatGoogleGenerativeAI (Gemini)}
    G --> H[Prompt Template: Instrucciones + Contexto + Pregunta]
    H --> G
    G --> I[Respuesta Generada por Gemini]
    I --> A
    subgraph Pre-procesamiento de Documentos
        J[Documentos PDF: Desenredando la ciencia] --> K[Loader: PyPDFLoader]
        K --> L[Texto por página]
        L --> M[Text Splitter: RecursiveCharacterTextSplitter]
        M --> E
    end
Ejemplo de creación del vectorstore (LangChain + Chroma):

Python
vectorstore = Chroma.from_documents(
    documents=fragmentos,           
    embedding=embeddings,           
    collection_name="documentos_empresa",
    persist_directory="./chroma_db"
)
Detalles técnicos 🛠️
Ingesta: PDF (ej. desde Google Drive) con PyPDFLoader.
Chunking: RecursiveCharacterTextSplitter (chunk_size=500, chunk_overlap=50).
Embeddings: intfloat/multilingual-e5-large (SentenceTransformerEmbeddings, calculados localmente).
Vector DB: ChromaDB (persistencia en ./chroma_db).
Retriever: vectorstore.as_retriever(search_kwargs={"k": 3}).
Orquestación: LangChain (RetrievalQA.from_chain_type).
Generación: ChatGoogleGenerativeAI via Gemini-2.5-flash (requiere GEMINI_API_KEY).
Interfaz: Notebook (Jupyter/Colab). Se puede migrar a Streamlit o Gradio.
Stack tecnológico 🧰
LLM (generación): Gemini-2.5-flash (ChatGoogleGenerativeAI)
Embeddings: intfloat/multilingual-e5-large
Vector DB: ChromaDB (chromadb)
Framework: LangChain
UI: Jupyter Notebook (recomendado). Opcional: Streamlit / Gradio
Otras librerías: sentence-transformers, pypdf, langchain-community / langchain_community
Corpus de documentos 🗂️
Dominio: Educación — lógica, filosofía e historia de la ciencia
Fuente: Buacar, N. (2022) Compiladora. Desenredando la ciencia. Eudeba.
Formato: PDF (español)
Extensión: ~15 capítulos, 410 páginas
Instalación y uso local ⚙️
Requisitos

Python 3.12.12+
GEMINI_API_KEY (API key de Gemini) — solo para la generación
Clonar repo:

bash
git clone https://github.com/RocioSK-collab/T-cnicas-de-Procesamiento-del-habla-.git
cd T-cnicas-de-Procesamiento-del-habla-/TP2_Stefanazzi_Kondolf_Rocio
Crear entorno virtual:

bash
python -m venv venv
source venv/bin/activate    # Linux/Mac
venv\Scripts\activate       # Windows
Instalar dependencias:

Si tienes requirements.txt:
bash
pip install -r requirements.txt
Si no, instalar dependencias mínimas:
bash
pip install langchain langchain-google-genai langchain-chroma chromadb sentence-transformers pypdf
pip install -U langchain-community langchain_community
Variables de entorno:

bash
# .env o exportar temporalmente
GEMINI_API_KEY=tu_api_key
Procesamiento de documentos (notebook):

Coloca el PDF en la ruta configurada (ej. Google Drive) o adapta la ruta en el notebook.
Pasos: PyPDFLoader → RecursiveCharacterTextSplitter (500/50) → embeddings (intfloat/multilingual-e5-large) → Chroma.from_documents(..., persist_directory="./chroma_db").
Ejecutar notebook:

bash
jupyter notebook "TP_FINAL_NLP_ (5).ipynb"
# o ejecutar todo:
jupyter nbconvert --to notebook --execute "TP_FINAL_NLP_ (5).ipynb" --ExecutePreprocessor.timeout=600 --output executed.ipynb
Ejecutar app (opcional — si creas app.py con Streamlit):

bash
streamlit run app.py
Ejemplos de consultas sugeridas 📝
¿Cuándo un paradigma logra constituirse en ciencia normal, cuál es el trabajo de los científicos?
¿Cuáles son las condiciones para que una teoría pueda considerarse científica según Popper?
¿Cuándo un argumento es válido?
Comparame las concepciones de Darwin y Lamarck.
Decisiones de diseño 🧩
Gemini-2.5-flash: equilibrio entre accesibilidad y calidad de generación.
chunk_size = 500 & overlap = 50: buen balance entre granularidad y coherencia.
top-k = 3: suficiente para cubrir contexto sin sobrecargar la generación.
Limitaciones conocidas ⚠️
ChromaDB indexa texto; no hay soporte nativo para recuperar tablas, gráficos o imágenes con estructura.
Dependencia de recursos en Colab (GPU / memoria).
La implementación principal está en notebook; no hay UI de producción incluida.
Posibles conflictos de versión entre paquetes (ej. langchain vs langchain-core).
Mejoras futuras 🚀
Indexar y recuperar tablas/figuras (herramientas multimodales).
Desplegar una UI persistente (Streamlit/Gradio + backend).
Añadir metadatos y segmentación por capítulo/sección para mejorar trazabilidad.
Métricas/validación automática de fidelidad entre respuesta y fuentes.
Troubleshooting 🛠️
API key inválida: revisar documentación de Gemini — https://ai.google.dev/gemini-api/docs
Out of memory (Colab): usar otra cuenta/instancia, reducir batch size o usar máquinas con más memoria.
Conflictos de versión: crear requirements.txt con versiones fijas o usar un entorno limpio.
Estructura propuesta del proyecto
Code
├── app.py                  # (opcional) Aplicación Streamlit
├── ingest_documents.py     # Script de ingesta y procesamiento
├── utils.py                # Funciones auxiliares
├── requirements.txt        # Dependencias
├── README.md               # Este archivo
├── .env.example            # Plantilla de variables de entorno
├── data/                   # PDFs y documentos fuente
│   └── Desenredando_la_ciencia.pdf
├── chroma_db/              # Base de datos vectorial (generada)
└── notebooks/              # Notebooks (TP_FINAL_NLP_ (5).ipynb)
Autor ✍️
Rocío Stefanazzi Kondolf
Trabajo Integrador N°2 — Técnicas de Procesamiento del Habla (IFTS 24)
Profesor: Matías Barreto — Año: 2025

