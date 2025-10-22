# meme-interpretation

Interpret, filter, and evaluate Indonesian memes using vision-capable LLMs. This project generates hidden/contextual meanings for memes, flags unsafe content (pornographic), and compares outputs across multiple models.

## What this repository does
- **Interpretation**: Produces Indonesian-language explanations of meme meaning using image + OCR text.
- **Safety filtering**: Classifies memes as Safe/Unsafe (pornographic content focus).
- **Evaluation**: Measures agreement across models via multilingual sentence embeddings and cosine similarity.

## Repository layout
- **`Data/`**
  - `Filtered 230/`: 230 images used for interpretation.
  - `New500/`: ~566 images used for safety filtering.
  - `New230SHEETS.csv`: CSV with columns `id, image_name, ocr_txt` for 230 samples.
  - `filter.ipynb`: Notebook that runs the Safe/Unsafe classifier workflow on `New500/` with `Prompt/filter.md`.
- **`Interpreter/`**
  - `interpreter-openai.ipynb`: Runs interpretation with `gpt-4.1` (OpenAI).
  - `interpreter-groq.ipynb`: Runs interpretation with `meta-llama/llama-4-scout-17b-16e-instruct` (Groq).
  - `interpreter-gemini.ipynb`: Runs interpretation with `gemini-2.5-pro` through an OpenAI-compatible endpoint.
- **`Prompt/`**
  - `interpreter.md`: Prompt template for interpretation (Indonesian, image + text, example, query placeholders).
  - `filter.md`: Prompt template for Safe/Unsafe classification with definitions and examples.
- **Root files**
  - `evaluation.ipynb`: Cleans and embeds model outputs, computes pairwise/average similarities, highlights low-agreement cases, saves a filtered CSV.
  - `requirements.txt`: Base Python dependencies used across notebooks.

## Setup
1) Create and activate a virtual environment
```bash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\\Scripts\\activate
```

2) Install dependencies
```bash
pip install -r requirements.txt
# Additional packages used in notebooks (not listed in requirements.txt):
pip install groq Sastrawi sentence-transformers scikit-learn
```

3) Prepare environment variables in a `.env` at the repository root
```bash
OPENAI_API_KEY=your_openai_key
GROQ_API_KEY=your_groq_key
GEMINI_API_KEY=your_gemini_key
```

### Data expectations
- Place images under `Data/Filtered 230/` (for interpretation runs) and `Data/New500/` (for safety filtering).
- Ensure `Data/New230SHEETS.csv` exists with columns `id, image_name, ocr_txt`. The notebooks assume the working directory is the repo root.

### Running the workflows
- **Interpretation with OpenAI** (`Interpreter/interpreter-openai.ipynb`)
  - Loads `Prompt/interpreter.md` and `Data/New230SHEETS.csv`.
  - Reads images from `Data/Filtered 230/` and sends prompt + image to `gpt-4.1`.
  - Extracts a clean textual interpretation (`output_extracted`) and saves `gpt4.1_results.csv`.

- **Interpretation with Groq** (`Interpreter/interpreter-groq.ipynb`)
  - Same pipeline; uses `meta-llama/llama-4-scout-17b-16e-instruct` via the `groq` SDK.
  - Saves `meta_maverick_results.csv`.

- **Interpretation with Gemini** (`Interpreter/interpreter-gemini.ipynb`)
  - Same pipeline; initializes the OpenAI client with `base_url="https://generativelanguage.googleapis.com/v1beta/openai/"` and `model="gemini-2.5-pro"`.
  - Saves `gemini_2.5_pro_results.csv`.

- **Safety filtering** (`Data/filter.ipynb`)
  - Loads `Prompt/filter.md` and iterates over `Data/New500/`.
  - Classifies each meme as Safe/Unsafe with `gpt-4.1`, saves `meme_classification_results.csv`, and prints counts.

- **Evaluation and agreement** (`evaluation.ipynb`)
  - Loads `gpt4.1_results.csv`, `meta_maverick_results.csv`, and `gemini_2.5_pro_results.csv`.
  - Cleans text (remove the word “meme”, punctuation, numbers), removes Indonesian stopwords (Sastrawi), stems, deduplicates words.
  - Embeds with `sentence-transformers` (`paraphrase-multilingual-MiniLM-L12-v2`), computes cosine similarities across models per meme, averages per row.
  - Reports averages and low-agreement cases; saves `Results/sentence_embed_results_filtered.csv`.

### Outputs
- `gpt4.1_results.csv`: Interpretations from OpenAI.
- `meta_maverick_results.csv`: Interpretations from Groq.
- `gemini_2.5_pro_results.csv`: Interpretations from Gemini.
- `meme_classification_results.csv`: Safe/Unsafe results for `Data/New500/`.
- `Results/sentence_embed_results_filtered.csv`: Evaluation outputs with similarities.

### Prompt templates
- `Prompt/interpreter.md` expects placeholders:
  - `{image_path}` → `image_name` from CSV
  - `{meme_text}` → `ocr_txt` from CSV
- `Prompt/filter.md` expects placeholders:
  - `{image_path}` → `image_name`
  - `{image_text}` → `ocr_txt`

### Notes and gotchas
- Keep the working directory at the repository root when running notebooks, as paths are relative.
- On Windows, avoid backslashes in string literals when passing paths in code; the notebooks already use correct joins for local runs.
- The Gemini notebook intentionally uses the OpenAI SDK with a custom `base_url` and `model="gemini-2.5-pro"`.
- Images are sent as base64 `data:image/jpeg;base64,...` within chat `image_url` content for all providers.
- API usage can be slow and costly. The loops include a `time.sleep(1)` to respect rate limits.

### Requirements
`requirements.txt` includes base libs:
```
openai
pandas
numpy
opencv-python
matplotlib
tqdm
python-dotenv
```
Notebooks also use: `groq`, `Sastrawi`, `sentence-transformers`, and `scikit-learn`.

### Quick start
1) Install dependencies and set `.env` keys.
2) Ensure images and CSV are placed under `Data/` as described.
3) Run one of the interpreter notebooks under `Interpreter/`.
4) Optionally run `Data/filter.ipynb` for Safe/Unsafe and `evaluation.ipynb` for cross-model agreement.

### License
If you plan to open-source this work, add a license file and reference it here.