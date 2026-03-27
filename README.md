# Gemini File Search Store Manager & RAG Chat

> **⚠️ Archive Notice (March 2026)**
>
> This project is **archived** and no longer actively maintained.
>
> The [Documents API](https://ai.google.dev/api/file-search/documents) — which
> allows listing, retrieving, and deleting individual documents within a File Search
> Store — already existed when this tool was developed (December 2025), but was
> overlooked during the design phase. The SQLite-based state management layer this
> tool provides is therefore unnecessary for most use cases.
>
> The code remains available as a reference for:
>
> - Blue/Green store refresh pattern
> - MD5-based deduplication for File Search uploads
> - Streamlit + SQLAlchemy integration example
>
> For the full story, see the [Zenn article (Japanese)](https://zenn.dev/yh_catalysis/articles/filesearchtoolmgmt-dev1).

A local Streamlit application for managing Google Gemini API [File Search Stores](https://ai.google.dev/gemini-api/docs/file-search) and conducting RAG (Retrieval-Augmented Generation) chats.

This tool provides a GUI to upload documents, manage vector store lifecycles (with Blue/Green refresh), and chat with your knowledge base using persistent history tracking.

![Python](https://img.shields.io/badge/Python-3.12%2B-blue)
![Streamlit](https://img.shields.io/badge/Streamlit-1.40%2B-FF4B4B)
![Gemini API](https://img.shields.io/badge/Google%20GenAI%20SDK-v1.0-4285F4)

## 🌟 Key Features

### 📄 Document Management

- **Smart Upload:** Upload PDFs, Code, and Text files. Automatically deduplicates based on MD5 hashes to save API costs.
- **Blue/Green Refresh:** Safely update your knowledge base. The tool creates a new Store, re-indexes local files, and swaps the ID only after success, ensuring zero downtime.
- **Local Persistence:** Keeps track of files and Store IDs in a local SQLite database (`gemini_store.db`) and preserves raw files in `./input_files/`.

### 💬 RAG Chat

- **Streaming Response:** Real-time text generation with `gemini-2.5-flash-lite` (configurable).
- **Persistent History:** Chat sessions are saved locally. You can switch between past conversations anytime.
- **Citations:** Automatically displays grounding sources (Web/File) referenced by the model.
- **Auto-Titling:** Automatically generates a concise title for new chat sessions based on the context.
- **Export:** Download chat history as JSON.

### ⚙️ Management

- **History Manager:** View and delete old chat sessions.
- **Robust Error Handling:** Automatic retries for API uploads and rate limit handling.

## 🚀 Installation

### Prerequisites

- Python 3.12+
- A Google Cloud Project with Gemini API enabled
- [uv](https://github.com/astral-sh/uv) (Recommended) or pip

### 1. Clone the Repository

```bash
git clone https://github.com/yh-catalysis/gemini_file_search_tool_mgmt.git
cd gemini_file_search_tool_mgmt
```

### 2. Install Dependencies

Using `uv` (Fast & Recommended):

```bash
uv sync
```

Or using standard `pip`:

```bash
pip install streamlit -r requirements.txt
```

### 3. Configuration

Create a `.env` file in the project root:

```env
# Required
GOOGLE_API_KEY="your-gemini-api-key"

# Optional
GEMINI_MODEL="gemini-2.5-flash-lite"
```

## 💻 Usage

Start the application:

```bash
uv run streamlit run app.py
# or
streamlit run app.py
```

### Navigation

Use the sidebar radio buttons to switch between modes:

1. **Documents:**
   - Select or Create a Store.
   - Drag & Drop files to upload.
   - Click **"🔄 Refresh Store"** if you have modified files locally or want to clean up the index.
2. **RAG Chat:**
   - Select a Store to chat with.
   - Create "✨ New Session" or resume history.
   - Export history via the button above the input bar.
3. **History Manager:**
   - View list of all chat sessions.
   - Delete specific histories.

## 🏗️ Architecture

### Local State (SQLite)

Since the Gemini API does not provide an easy way to list/manage individual file metadata within a Store, this app maintains a local `gemini_store.db` using SQLAlchemy.

- `store_records`: Manages Store IDs and names.
- `file_records`: Tracks file paths, MD5 hashes, and sync status.
- `chat_sessions` & `chat_messages`: Stores conversation history.

### Blue/Green Refresh Strategy

To allow editing of local files (e.g., Markdown) without breaking the live Store:

1. **Create:** A new File Search Store is created.
2. **Migrate:** Valid local files (from `./input_files/`) are uploaded to the new Store.
3. **Swap:** The app updates the DB to point to the new Store ID.
4. **Cleanup:** The old Store is deleted from the Gemini API.

## 📁 Directory Structure

```text
.
├── app.py                # Main Streamlit Application
├── gemini_store.db       # SQLite Database (Auto-generated)
├── input_files/          # Local file storage (Auto-generated)
├── .env                  # API Keys
└── pyproject.toml        # Dependencies
```

## 🤝 Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

## 📄 License

[MIT](https://choosealicense.com/licenses/mit/)
