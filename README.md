# Semantic Image Search

This project implements a semantic image search engine leveraging a combination of CLIP for image and text embeddings, a vector database (Qdrant) for efficient similarity search, and a Large Language Model (LLM) for intelligent query translation. The application provides both a FastAPI backend and a Streamlit user interface.

## Features

*   **Text-to-Image Search:** Search for images using natural language queries.
*   **Image-to-Image Search:** Find similar images by uploading an image.
*   **LLM-Powered Query Translation:** Enhances text search accuracy by rewriting user queries into optimized image captions (supports OpenAI and Google Generative AI).
*   **Qdrant Vector Database:** Efficiently stores and retrieves image embeddings.
*   **Scalable Architecture:** Built with FastAPI for the backend and Streamlit for a user-friendly frontend.

## Prerequisites

Before you begin, ensure you have the following installed on your system:

*   **Python 3.9+**: [Download Python](https://www.python.org/downloads/)
*   **Git**: [Download Git](https://git-scm.com/downloads)
*   **Google API Key**: Obtain a Google API Key for Google Generative AI (Gemini models) from the [Google Cloud Console](https://console.cloud.google.com/apis/credentials).
*   **Qdrant Cloud Account & API Key**: Set up an account on [Qdrant Cloud](https://cloud.qdrant.io/) and create an API key and a cluster.

## Setup Instructions

Follow these steps to get your project up and running:

1.  **Clone the Repository:**

    ```bash
    git clone https://github.com/Venumurala91/sementic-image-search.git
    cd sementic-image-search
    ```

2.  **Create and Activate a Virtual Environment:**

    It's recommended to use a virtual environment to manage project dependencies.

    ```bash
    python -m venv env
    # On Windows:
    .\env\Scripts\activate
    # On macOS/Linux:
    source env/bin/activate
    ```

3.  **Install Dependencies:**

    Install all required Python packages using pip:

    ```bash
    pip install -r requirements.txt
    ```

4.  **Configure Environment Variables:**

    Create a `.env` file in the root directory of your project (e.g., `sementic-image-search/.env`) and add the following:

    ```dotenv
    Google_API_Key="YOUR_GOOGLE_API_KEY"
    # Optional: If you want to use OpenAI models, uncomment and add your key
    # OPENAI_API_KEY="YOUR_OPENAI_API_KEY"

    # Qdrant Configuration
    QDRANT_API_KEY="YOUR_QDRANT_API_KEY"
    QDRANT_URL="YOUR_QDRANT_CLUSTER_URL" # e.g., "https://[cluster-id].us-east-1-1.aws.cloud.qdrant.io:6333"

    # Optional: HuggingFace Token if you are using HuggingFace models
    # HF_Token="YOUR_HF_TOKEN"
    ```
    Replace `"YOUR_GOOGLE_API_KEY"`, `"YOUR_QDRANT_API_KEY"`, `"YOUR_QDRANT_CLUSTER_URL"`, and optionally `"YOUR_OPENAI_API_KEY"` and `"YOUR_HF_TOKEN"` with your actual keys and URL.

## Running the Project

The project consists of a backend API (FastAPI) and a frontend UI (Streamlit). Both need to be run separately.

### 1. Start the Backend API

Open a new terminal, navigate to the project's root directory, and run the FastAPI application. This command will keep the backend running in this terminal:

```powershell
.\env\Scripts\python.exe -m uvicorn semantic_image_search.backend.main:app --host 0.0.0.0 --port 8000

or 

uvicorn semantic_image_search.backend.main:app --host 0.0.0.0 --port 8000
```
*(On macOS/Linux, replace `.\env\Scripts\python.exe` with `env/bin/python`)*

### 2. Start the Frontend UI

Open **another new terminal**, navigate to the project's root directory, and run the Streamlit application:

```powershell
.\env\Scripts\python.exe -m streamlit run semantic_image_search/ui/app.py

or 

streamlit run semantic_image_search/ui/app.py
```
*(On macOS/Linux, replace `.\env\Scripts\python.exe` with `env/bin/python`)*

This will open the Streamlit application in your default web browser.

## Image Ingestion (Populating Qdrant)

To add your images to the Qdrant database for searching, you need to run the ingestion script.

1.  **Ensure Backend is Running:** Make sure your backend API is already running (as described in "1. Start the Backend API" above).

2.  **Run the Ingestion Script:**
    Open a **new terminal** (separate from the backend and frontend terminals), navigate to the project's root directory, and run the `ingestion.py` script. You will need to modify the `if __name__ == "__main__":` block in `semantic_image_search/backend/ingestion.py` to specify the `root_folder` containing your images. For example:

    ```python
    # Inside semantic_image_search/backend/ingestion.py, at the very bottom:
    if __name__ == "__main__":
        indexer = IndexService()
        # Replace 'path/to/your/images' with the actual path to your image folder
        indexer.index_folder("D:/ai_projects/mlops_project_by_sunny/sementic-image-search/images")
        # You can also index a single image:
        # indexer.index_image("path/to/single/image.jpg", category="my_category")
    ```
    After modifying `ingestion.py` with your image folder path, run it from the project root:

    ```powershell
    .\env\Scripts\python.exe semantic_image_search/backend/ingestion.py
    ```
    *(On macOS/Linux, replace `.\env\Scripts\python.exe` with `env/bin/python`)*

## Workflow

The following flowchart illustrates the high-level workflow of the Semantic Image Search project:

```mermaid
graph TD
    A[User Input] --> B{Input Type?};
    B -- Text Query --> C[Query Translation (LLM)];
    C --> D[Generate Text Embedding (CLIP)];
    B -- Image Upload --> E[Generate Image Embedding (CLIP)];
    D --> F[Vector Search (Qdrant)];
    E --> F;
    F --> G{Retrieve Top-K Similar Images};
    G --> H[Display Results (Streamlit UI)];
```

### Workflow Details:

1.  **User Input:** The process begins with the user interacting with the Streamlit frontend, providing either a text-based query or uploading an image.

2.  **Query Translation (for Text Queries Only):**
    *   If the input is a text query, it's sent to the backend's `QueryTranslator`.
    *   An LLM (configured in `semantic_image_search/backend/query_translator.py`, e.g., Google's `gemini-1.5-flash` or OpenAI's `gpt-4o-mini`) rewrites the user's natural language query into a concise, descriptive image caption. This step optimizes the query for better retrieval by the CLIP model.

3.  **Embedding Generation (CLIP Model):**
    *   **For translated text queries:** The rewritten caption is processed by the CLIP (Contrastive Language-Image Pre-training) model. CLIP converts this text into a high-dimensional numerical vector, known as a text embedding.
    *   **For image uploads:** The uploaded image is directly processed by the CLIP model, which generates a corresponding image embedding.
    *   The CLIP model ensures that both text and image data are represented in the same vector space, where semantically similar items (e.g., a "cat" text query and an image of a cat) have closely located embeddings.

4.  **Vector Search (Qdrant):**
    *   The generated embedding (either from text or image) is sent to the **Qdrant vector database**.
    *   Qdrant efficiently searches its stored collection of image embeddings for those that are most similar to the query embedding. This similarity is typically measured using cosine distance in the vector space.

5.  **Retrieve Top-K Similar Images:**
    *   Qdrant returns the metadata (e.g., file paths, categories, filenames) of the top-K most similar images from its database. The value of 'K' determines how many results are retrieved.

6.  **Display Results (Streamlit UI):**
    *   The backend sends this retrieved image metadata to the Streamlit frontend.
    *   The Streamlit UI then uses the provided image paths to load and display the relevant images to the user, completing the search process.
