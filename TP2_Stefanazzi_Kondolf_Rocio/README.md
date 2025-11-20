#Sistema RAG para consultas bibliográficas
Descripción
Mi sistema RAG permite que docentes de IPC puedan chequear información, parafrasear ideas, buscar información determinada, comparar autores consultando información del manual de lógica, filosofía e historia de la ciencia titulado Desenredando la ciencia (Buacar, Editorial Eudeba, 2022) mediante búsqueda semántica con ChromaDB y generación con Gemini.
Desarrollado como Trabajo Integrador N°2 para la materia Técnicas de Procesamiento del Habla (IFTS 24).


## Problema que Resuelve

El problema que el proyecto intenta resolver surge de la necesidad de un mecanismo de verificación de la información a la hora de formular preguntas específicas para exámenes asociadas a ciertos libros de textos.
Se busca crear un asistente de IA para el apoyo docente en la formulación de exámenes. En particular, en tareas como chequear la información a partir de la fuente, parafrasear ideas, buscar información determinada, comparar autores, etc.
Una vez probado por los docentes, el proyecto podría ser escalado para realizar consultas sobre dudas bibliográficas por parte de los estudiantes lo cual sería una forma de amplificar el trabajo docente de revisión de consultas y llegar a mayor cantidad de estudiantes.
La solución al problema es utilizar un sistema RAG (Generación aumentada por recuperación) que permita utilizar la potencia de los LLMs sin que se produzcan alucinaciones. El sistema RAG es muy útil para limitar a los modelos para que respondan en base a los textos con los que los que fueron “alimentados” sin considerar posibles respuestas que utilicen información por fuera de los textos específicos (basándose en información de entrenamiento, por ejemplo). En ese sentido garantizan una fiabilidad en la información.
La sigla hace referencia a tres palabras. La R de “recuperación” remite a la búsqueda del sistema en la base de datos vectorial. La A de “aumentada” combina la pregunta del usuario con la información para generar un prompt interno al cual se lo enriquece para consultarle al modelo. La G de “generación” hace referencia al proceso por el cual el modelo de lenguaje recibe el prompt y se basa en los fragmentos de texto asociados a él dados por las bases de datos vectoriales.
El proceso consiste en fragmentar los textos en chunks, vectorizar los textos (embeddings) y almacenarlos en bases vectoriales. Se crearán embbedings a partir de los textos específicos de la cátedra y se guardarán en bases de datos vectoriales utilizando chroma. Se utilizará langchain como framework que permite acceder los datos e interactuar con el usuario, articulando el proceso. Se crearán templates de preguntas que servirán como generadores de prompts. Se creará una interfaz de usuario con gradio. El LLM a utilizar será Gemini 2.5.

## Arquitectura del Sistema

### Pipeline RAG
Flujo de trabajo del sistema:
   1. Usuario hace una pregunta
   2. El sistema busca los 3 fragmentos más relevantes (LOCAL)
   3. Gemini lee esos fragmentos y genera una respuesta (API)
   4. Se devuelve la respuesta + documentos fuente

1. **Ingesta**: Se cargó el documento a través de googledrive en formato pdf
2. **Chunking**: chunks de 500 caracteres con overlap de 50
3. **Embeddings**: intfloat/multilingual-e5-large
4. **Almacenamiento**: ChromaDB con vectorstore = Chroma.from_documents(
    documents=fragmentos,   embedding=embeddings, collection_name="documentos_empresa",
    persist_directory="./chroma_db"
5. **Retrieval**:  top-k 3
6. **Generation**: Gemini-2.5-flash
7. **Interfaz**: Streamlit


### Diagrama de Flujo
graph TD
    A[Usuario: Pregunta] --> B[Sistema RAG: RetrievalQA]
    B --> C{Retriever: Búsqueda de Embeddings}
    C --> D[Vector Store: ChromaDB]
    D --> E[Embeddings: intfloat/multilingual-e5-large]
    E --> C
    C --> F[Fragmentos de Documentos Relevantes]
    F --> G{LLM: ChatGoogleGenerativeAI}
    G --> H[Prompt Template: Instrucciones + Contexto + Pregunta]
    H --> G
    G --> I[Respuesta Generada por Gemini]
    I --> A
    subgraph Pre-procesamiento de Documentos
        J[Documentos PDF: Desenredando la ciencia] --> K[Loader: PyPDFLoader]
        K --> L[Texto Completo por Página]
        L --> M[Text Splitter: RecursiveCharacterTextSplitter]
        M --> E
    end



## Stack Tecnológico

- **LLM**: Gemini-2.5-flas]
- **Embeddings**: intfloat/multilingual-e5-large
- **Vector Database**: ChromaDB
- **Orquestación**: LangChain
- **Interfaz**: Streamlit


## Corpus de Documentos

- **Dominio**: Educación en lógica, filosofía e historia de la ciencia
- **Cantidad**: un manual de 15 capítulos y 410 págs.
- **Fuente**: Buacar, N. (2022) Compiladora. Desenredando la ciencia. Buenos Aires, Eudeba.
- **Formato**: PDF
- **Idioma**: español

## Instalación y Uso Local

Pasos de Instalación
1) Clonar el repositorio:

bash
git clone https://github.com/RocioSK-collab/T-cnicas-de-Procesamiento-del-habla-.git
cd T-cnicas-de-Procesamiento-del-habla-/TP2_Stefanazzi_Kondolf_Rocio

2)Crear entorno virtual:

bash
python -m venv venv
source venv/bin/activate    # En Windows: venv\Scripts\activate

3) Instalar dependencias:
-Si existe requirements.txt en tu flujo, instala con:

bash
pip install -r requirements.txt

-Si NO hay requirements.txt (el notebook instala las librerías directamente), instala las librerías usadas en el notebook:

bash
pip install langchain langchain-google-genai langchain-chroma chromadb sentence-transformers pypdf -q
Dependencias adicionales que aparecen en el notebook (opcional / según necesidad)
pip install -U langchain-community -q
pip install langchain_community -q
Nota: el notebook muestra mensajes de posibles conflictos entre versiones de paquetes (por ejemplo entre langchain y langchain-core). Si aparecen errores, crea un requirements.txt con versiones fijadas o instala en un entorno limpio.

4) Configurar variables de entorno (si aplica):

bash
Crear archivo .env con:
GEMINI_API_KEY=tu_api_key

O exportarla temporalmente en Linux/Mac:
export GEMINI_API_KEY=tu_api_key

En Windows PowerShell:
setx GEMINI_API_KEY "tu_api_key"
Notas importantes extraídas del notebook:

El notebook busca GEMINI_API_KEY en los secretos de Colab o en la variable de entorno local.
Solo la generación final usa la API de Gemini; los embeddings se calculan localmente.

5) Si es primera vez: Procesar documentos:
- El notebook carga un PDF desde Google Drive en la ruta utilizada en el ejemplo: /content/drive/MyDrive/IPC/Buacar (2022) Desenredando la ciencia - Completo.pdf
- Asegúrate de colocar tu PDF en esa ubicación o adapta la ruta en el notebook/script.
- Opciones para procesar:
  *Ejecutar el notebook interactivo (recomendado para validar paso a paso):

  bash
  jupyter notebook "TP_FINAL_NLP_ (5).ipynb"
     ó
  jupyter lab

  *Ejecutar el notebook por línea de comando (ejecuta todas las celdas):

  bash
  jupyter nbconvert --to notebook --execute "TP_FINAL_NLP_ (5).ipynb" --ExecutePreprocessor.timeout=600 --output         executed.ipynb

Si prefieres automatizar con un script (ingest_documents.py), implementa en ese script los pasos del notebook:
cargar PDF con PyPDFLoader (langchain.document_loaders), dividir con RecursiveCharacterTextSplitter (chunk_size=500, chunk_overlap=50), calcular embeddings locales (modelo intfloat/multilingual-e5-large), crear/guardar Chroma DB: Chroma.from_documents(..., persist_directory="./chroma_db").

6) Ejecutar la aplicación:
En este repositorio la implementación del sistema RAG está en el notebook TP_FINAL_NLP_ (5).ipynb (no hay app.py/Streamlit incluido).
Para usarlo:
Abre el notebook en Jupyter/Colab y ejecuta las celdas (desde "Configuración del entorno" hasta "Ensamblar el Sistema RAG" y pruebas).
Si quieres exponer una UI (Streamlit u otra), tendrás que convertir la lógica del notebook en app.py. Ejemplo (si creas app.py):

bash
streamlit run app.py
(Nota: app.py no está incluido en este repo; crearás ese archivo si deseas una interfaz web.)

7) Abrir en navegador:
Si ejecutas el notebook en Jupyter, abre la URL que te provea Jupyter (por defecto: http://localhost:8888).
Si implementas una app Streamlit (no incluida por defecto), la interfaz por defecto sería: http://localhost:8501
Resumen técnico y detalles extraídos del notebook (útiles para el README)

Librerías principales usadas: langchain, langchain-google-genai (ChatGoogleGenerativeAI), langchain-chroma / chromadb, langchain_community, sentence-transformers, pypdf.
Pipeline reproducido en el notebook:

Carga del PDF (PyPDFLoader). En el notebook se monta Google Drive en Colab para acceder al PDF de ejemplo.
División en fragmentos con RecursiveCharacterTextSplitter (chunk_size=500, chunk_overlap=50).
Embeddings calculados localmente usando el modelo multilenguaje intfloat/multilingual-e5-large (SentenceTransformerEmbeddings) — evita consumo de API para embeddings.
Guardado en ChromaDB (persist_directory="./chroma_db") con Chroma.from_documents(...).

Configuración del LLM generador: ChatGoogleGenerativeAI (Gemini) con modelo "models/gemini-2.5-flash" y temperature=0.1 (requiere GEMINI_API_KEY).
Ensamblado del sistema RAG con RetrievalQA.from_chain_type(..., retriever=vectorstore.as_retriever(search_kwargs={"k":3}), return_source_documents=True).

Función de prueba hacer_pregunta(pregunta) que devuelve respuesta generada + documentos fuente consultados.
Parámetros clave usados en el notebook:
chunk_size: 500, chunk_overlap: 50
embeddings model: intfloat/multilingual-e5-large
chroma persist directory: ./chroma_db
retrieval k: 3
Gemini model: models/gemini-2.5-flash (usa GEMINI_API_KEY)


### Prerrequisitos

- Python 3.12.12 +
- API key correspondiente
  

### Pasos de Instalación

```
├── app.py                  # Aplicación Streamlit principal
├── ingest_documents.py     # Script de ingesta y procesamiento
├── utils.py                # Funciones auxiliares
├── requirements.txt        # Dependencias
├── README.md              # Este archivo
├── .env.example           # Template de variables de entorno
├── data/                  # Documentos fuente
│   └── [tus documentos]
├── chroma_db/             # Base de datos vectorial (generada)
└── notebooks/             # Notebooks de exploración (opcional)
```

## Ejemplos de Consultas

Probá estas consultas de ejemplo:

1. ¿Cuándo un paradigma logra constituirse en ciencia normal, cuál es el trabajo de los científicos?
2. ¿Cuáles son las condiciones para que una teoría pueda considerarse científica según Popper?
3. ¿Cuándo un argumento es válido?
4. ¿Me hacés una compración entre las concepciones de Darwin y Lamarck?
   

## Decisiones de Diseño

### ¿Por qué elegí Gemini-2.5-flash?
Porque es de fácil acceso. 

### ¿Por qué 500 caracteres para cada chunk con 50 de overlap?
La cantidad de 500 caracteres es un tamaño lo suficientemente pequeño para poder mantener precisión temática. La superposición de 50 caracteres es útil y suficiente para mantener la coherencia entre chunks.

### ¿Por qué top-k = 3?
Porque la búsqueda de 3 fragmentos relevantes es suficiente para la extensión del material.


## Limitaciones Conocidas

- Limitación 1: la base d datos vectoriales se limita a texto, tendría que utilizar alguna que me permita recuperar tablas, gráficos e impagenes.
- Limitación 2: Tiene limitación el uso del GPU de google
- Limitación 3: Se dificulta generar una interfaz duradera desde colab con streamlit
  
## Mejoras Futuras

- [1] Utilizar una base de datos que capturen gráficos y tablas
- [2] Cragar guías de ejercicios y modelos de exámenes para crear ejercicios de manera automatizada
- [4] Desarrollar el modelo a través de un entorno local para no tener los problemas que surgieron de trabajar en la nube

## Troubleshooting

### Error: "API key inválida"
Dirigirse a https://ai.google.dev/gemini-api/docs y revisar la documentación actualizada

### Error: "Out of memory"
Copiar el cuaderno en otra cuenta de gmail y generar una nueva API

Errores comunes que encontraste: dificultades para generar la interfaz con streamlit, se quebaba tildada. Limitaciones en el uso del GPU de google. Cambios, actualizaciones de los modelos de Google y caída del sistema.

## Autor

Rocío Stefanazzi Kondolf

Trabajo Integrador N°2
Materia: Técnicas de Procesamiento del Habla 
Institución: IFTS 24 - Tecnicatura Superior en Ciencias de Datos e IA
Profesor: Matías Barreto
Año: 2025

