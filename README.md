Multiple PDF Q&A (RAG) Notebook
===============================

# My Project Name

[![Python 3.x](https://img.shields.io/badge/Python-3.x-blue.svg)](https://www.python.org/downloads/)
![Jupyter Notebook](https://img.shields.io/badge/Notebook-Jupyter-orange)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
![Ollama](https://img.shields.io/badge/LLM-ollama%2Fllama2-green)
![Qdrant](https://img.shields.io/badge/Vector%20DB-Qdrant-green)
![Docker](https://img.shields.io/badge/Docker-blue)

This repository demonstrates how to build a Retrieval-Augmented Generation (RAG) workflow using:

-   Loading **all PDFs** in a `data/` directory (via `DirectoryLoader` and `PyPDFLoader`).
-   Splitting PDFs into chunks using `RecursiveCharacterTextSplitter`.
-   Creating/storing vector embeddings in Qdrant for semantic search.
-   Querying a local or remote LLM (e.g., `ollama/llama2`) to answer questions based on retrieved context from Qdrant.

Table of Contents
-----------------

-   [Prerequisites](Prequisites)
-   [Installation](Installation)
-   [Data Preparation](Data-Preparation)
-   [Usage](Usage)
-   [Qdrant Setup](Qdrant-Setup)
-   [Notebook/Script Outline](Notebook/Script-Outline)
-   [Acknowledgments](Acknowledgments)
-   [License](License)

* * * * *

Prerequisites
-------------

1.  **Python 3** (tested with Python 3.x).
2.  **Docker** installed (if you want to run Qdrant locally).
3.  [Optional] A GPU-compatible environment (if you plan on running large models or speeding up embeddings). The code automatically falls back to CPU if no GPU is detected.

* * * * *

Installation
------------

1.  **Clone this repository** (or place the notebook/script files in your desired folder).

2.  Create a virtual environment (highly recommended):

    ```
    python3 -m venv venv
    source venv/bin/activate  # On macOS/Linux
    # or .\venv\Scripts\activate on Windows

    ```

3.  **Install dependencies** (some can be installed via the first cell in the notebook, or run manually in your terminal):

    ```
    pip install -qU langchain-text-splitters
    pip install qdrant-client
    pip install pypdf
    pip install python-dotenv
    pip install sentence-transformers
    pip install litellm
    pip install -qU langchain_community pypdf

    ```

    > **Note**: Depending on your system, you may need to install PyTorch (or an equivalent) separately to match your CUDA version.

4.  [Optional] If you want to run `ollama/llama2` locally, install [Ollama](https://github.com/jmorganca/ollama) and ensure it's running. Otherwise, adjust the script to use your preferred LLM or endpoint.

* * * * *

Data Preparation
----------------

-   **Put one or more PDFs** in the `data/` directory.
    -   The script loads **all** files matching `*.pdf` from that folder.
    -   By default, the code uses `DirectoryLoader` and `PyPDFLoader` to process them.
    -   If you see warnings like `Ignoring wrong pointing object ...`, they usually indicate minor PDF structure issues. They typically do *not* prevent extraction, so you can ignore them unless you notice missing text.

* * * * *

Usage
-----

1.  **Start Qdrant** (see [Qdrant Setup]).

2.  **Launch the Jupyter notebook** (or run the Python script) in your environment:

    ```
    jupyter notebook
    # or
    python your_script_name.py

    ```

3.  The script will:

    1.  **Load PDFs** from `data/` using `DirectoryLoader`.
    2.  **Split** the text into chunks using `RecursiveCharacterTextSplitter`.
    3.  **Compute embeddings** for each chunk (using `BAAI/bge-small-en-v1.5` by default).
    4.  **Store** these chunks + embeddings in a Qdrant collection (named `qa_index` by default).
    5.  Provide a **search** function that:
        -   Embeds your query text.
        -   Retrieves top-k similar chunks from Qdrant.
    6.  **Call the LLM** (here, `ollama/llama2`) to provide an answer using the retrieved context.
4.  **Asking questions**:

    -   Modify the `question` variable (e.g., `"Which are the 12 Factors?"`) in the script or notebook cell.
    -   Run the last cell(s) to see the final answer.
    -   The answer will be streamed from your chosen LLM, with references from the newly-ingested PDFs.
5.  **Clearing the database**:

    -   To wipe a single collection:

        ```
        client.delete_collection("qa_index")

        ```

    -   Or to remove *all* collections:

        ```
        collections = client.get_collections().collections
        for coll_info in collections:
            client.delete_collection(coll_info.name)

        ```

    -   If running Qdrant in Docker, removing the container/volume also fully clears stored data.

* * * * *

Qdrant Setup
------------

This example assumes a local Qdrant server running at `http://localhost:6333`. To spin it up quickly with Docker:

```
docker pull qdrant/qdrant
docker run -p 6333:6333\
    -v $(pwd)/qdrant_storage:/qdrant/storage\
    qdrant/qdrant

```

-   This command maps port `6333` and mounts `qdrant_storage` in your current directory to store data persistently.
-   Verify Qdrant is reachable at <http://localhost:6333/collections>.

* * * * *

Notebook/Script Outline
-----------------------

1.  **Installation/Import**\
    Installs or imports required libraries.

2.  **Load PDFs**\
    Uses `DirectoryLoader(path="data", glob="*.pdf", loader_cls=PyPDFLoader)` to load *all PDFs* in `data/`.

3.  **Text Splitting**\
    Applies `RecursiveCharacterTextSplitter` to chunk long PDF texts.

4.  **Embedding**\
    Creates embeddings using `SentenceTransformer("BAAI/bge-small-en-v1.5")` (on GPU if available).

5.  **Qdrant**

    -   Connects to `http://localhost:6333`.
    -   Creates (or deletes + recreates) a `COLLECTION_NAME` collection.
    -   Uploads chunk vectors + payload to Qdrant.
6.  **Search Function**\
    Accepts a query, returns top-k similar chunks (vectors) from Qdrant.

7.  **LLM Query**

    -   Combines top chunks into a single "context" string.
    -   Passes the context + user question to the LLM (via [litellm](https://github.com/mitchellharvey/litellm)).
    -   Streams the final response in the notebook/terminal.
8.  **Example Query**\
    Demonstrates how to ask questions like "Which are the 12 Factors?" and retrieve an answer from the newly ingested PDFs.

* * * * *

Acknowledgments
---------------

Many thanks to **Two Set AI** for providing guidance on implementing a RAG system, including code snippets and a tutorial video. You can check out their tutorial here:

-   [Two Set AI - YouTube Tutorial](https://www.youtube.com/watch?v=FKmjT93D50U)

* * * * *

License
-------

This project is licensed under the MIT License. See the license file for details (or adapt to whichever license you prefer).

* * * * *

Feel free to modify any sections to fit your specific use case or naming conventions. Enjoy exploring your documents with Qdrant + an LLM!