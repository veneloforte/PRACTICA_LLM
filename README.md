# Asistente Turistico de Tenerife (RAG + Gemini + LangChain)

## Descripcion
Este proyecto implementa un asistente turistico sobre Tenerife basado en Retrieval-Augmented Generation (RAG). El sistema combina:

- Carga y fragmentacion de contenido desde un PDF.
- Embeddings de Google Gemini para indexar conocimiento.
- Un vector store FAISS para recuperacion semantica.
- Un modelo de chat Gemini integrado con LangChain para responder preguntas y usar herramientas (Function Calling), como una funcion simulada de meteorologia.

El desarrollo principal esta en el notebook [GeminiLLM.ipynb](GeminiLLM.ipynb).

## Estructura del repositorio
- [GeminiLLM.ipynb](GeminiLLM.ipynb): notebook principal con pipeline RAG, evaluacion y analisis.
- [data/](data/): datos de entrada (incluye el PDF de Tenerife).
- [docs/](docs/): documentacion adicional y enunciado.
- [requirements.txt](requirements.txt): dependencias del proyecto.
- [geminiapi.env](geminiapi.env): archivo de variables de entorno (API key).

## Requisitos previos
- Python 3.10 o superior.
- Cuenta de Google AI Studio / Gemini Developer API.
- Clave API valida (GOOGLE_API_KEY).
- Jupyter Lab (o Jupyter Notebook) para ejecutar el notebook.

## Instalacion
1. Clonar el repositorio:

```bash
git clone <URL_DEL_REPOSITORIO>
cd PRACTICA
```

2. Crear y activar entorno virtual:

Windows (PowerShell):
```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Linux/macOS:
```bash
python3 -m venv .venv
source .venv/bin/activate
```

3. Instalar dependencias:

```bash
pip install -r requirements.txt
```

## Configuracion de variables de entorno
Crea un archivo llamado `geminiapi.env` en la raiz del proyecto con este contenido:

```env
GOOGLE_API_KEY="TU_API_KEY_AQUI"
```

Nota: En algunos enunciados aparece `gminiapi.env`; en este proyecto el notebook carga `geminiapi.env`.

Opcionalmente, tambien puedes usar `.env` con la misma variable.

## Ejecucion
1. Activar el entorno virtual (si no esta activo).
2. Lanzar Jupyter Lab:

```bash
jupyter lab
```

3. Abrir [GeminiLLM.ipynb](GeminiLLM.ipynb).
4. Ejecutar las celdas en orden de arriba a abajo.

## Flujo funcional resumido
1. Carga del PDF turistico de Tenerife.
2. Division en chunks (1000 chars, overlap 200).
3. Generacion de embeddings (`models/gemini-embedding-001`).
4. Indexacion y recuperacion semantica con FAISS.
5. Generacion de respuesta con modelo Gemini via LangChain.
6. Function Calling para consulta meteorologica simulada.

## Solucion de problemas rapida
- Error de API key: verifica que `GOOGLE_API_KEY` exista en `geminiapi.env` y ejecuta primero la celda de preparacion.
- Error de modulos (`ModuleNotFoundError`): confirma el kernel del notebook en `.venv` y reinstala dependencias.
- Error de modelo no encontrado: usa un modelo disponible para tu API key (por ejemplo, `gemini-flash-latest`).

## Entregables
- README del repositorio: este archivo.
- Informe final tecnico: [INFORME_FINAL.md](INFORME_FINAL.md).
