# PromptSage

PromptSage is a local Streamlit application for reviewing prompts with simple, explainable NLP and estimating their size with tiktoken. It reports text statistics, highlights frequent keywords and configured filler phrases, and proposes a conservative rule-based rewrite that you can edit before using.

## Problem statement

Prompts can contain unnecessary wording, and a writer may not know how their text is split into model tokens. PromptSage makes basic text properties and token counts visible, then suggests a small set of transparent edits. It uses no LLM API, paid service, database, authentication, or API key.

## Objectives and features

- Show character, word, sentence, unique-word, reading-time, and selected-encoding token counts.
- Identify frequent meaningful words, filler phrases, and repeated words.
- Apply deterministic phrase substitutions and whitespace cleanup while retaining other text such as numbers and constraints.
- Compare original and editable optimized text by tokens, words, and characters.
- Explain each NLP stage for a college demonstration.

## Technology stack

- Python 3.10+
- Streamlit for the interface
- NLTK's resource-free Treebank word tokenizer
- tiktoken for exact counts under `cl100k_base` or `o200k_base`
- Python `re` and `collections.Counter` for phrase checks and frequencies

## NLP concepts

**Tokenization** splits text into units for analysis. The analyzer uses NLTK's Treebank tokenizer to form word-like units without needing downloaded NLTK corpora. Separately, tiktoken uses encoding-specific byte-pair encoding (BPE) to split text into model token IDs. These are different tokenization schemes: whitespace-delimited words and model tokens are not interchangeable.

**Stopword filtering** removes common English words only from keyword frequency results. It never removes words from the user's prompt. A small built-in stopword list avoids runtime downloads and internet dependence.

**Rule-based optimization** applies a fixed list of regular-expression replacements and whitespace normalization. It is not a trained machine-learning model and cannot guarantee meaning preservation. Review the editable result, especially any constraints, before using it.

Sentence count uses punctuation boundaries (`.`, `!`, `?`) and line breaks as a practical heuristic. Reading time assumes 200 words per minute. Both are estimates rather than linguistic guarantees.

## Architecture

```text
app.py ──> nlp_analyzer.py   (word statistics and phrase insights)
      ├──> optimizer.py      (safe, deterministic rewrite rules)
      └──> token_counter.py  (cached tiktoken encodings and comparison)
```

## Project structure

```text
PromptSage/
├── app.py
├── nlp_analyzer.py
├── optimizer.py
├── token_counter.py
├── requirements.txt
├── README.md
├── .gitignore
└── tests/
    ├── test_nlp_analyzer.py
    ├── test_optimizer.py
    └── test_token_counter.py
```

## Installation on Windows

Open PowerShell in the project folder and create a virtual environment:

```powershell
py -3.10 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

If PowerShell blocks activation, run `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` in that terminal and activate again.

## Run the application

```powershell
python -m streamlit run app.py
```

Streamlit prints a local URL (usually `http://localhost:8501`) to open in your browser. The first use of a tiktoken encoding may download its tokenizer data; tokenization initialization errors are surfaced in the app rather than replaced by an estimate. NLTK does not download corpora at runtime.

## Example

Input:

> Can you please summarize the causes of the French Revolution in 150 words. Include the years 1789 and 1799.

The analyzer reports counts and meaningful keywords such as “French,” “Revolution,” and “causes.” The rule-based suggestion removes “Can you please” while keeping the 150-word limit and both years. The exact token figures depend on the selected encoding.

## How optimization works

The optimizer applies only the visible patterns in `REPLACEMENTS` in `optimizer.py`, then collapses duplicate horizontal whitespace and trims outer whitespace. It does not delete general stopwords or generate new content. The before-and-after comparison counts both versions with the same selected tiktoken encoding. Savings can be negative if an edit or manual change increases the count; the UI displays that honestly. A percentage is zero when the original has zero tokens.

## Limitations and future improvements

- Sentence splitting is heuristic and abbreviations may be counted imperfectly.
- The built-in stopword list is compact, English-focused, and not language detection.
- Phrase detection only finds configured phrases; it does not judge semantic redundancy.
- Rule-based rewrites can affect meaning, so human review remains necessary.
- Token counts are exact for the selected encoding, but may differ from the tokenizer used by a particular AI model.
- Future work could add language-aware analysis, user-configurable rules, and a richer visualization of word frequencies.

## Likely viva questions and answers

**1. What is the difference between tokenization and stopword removal?**  
Tokenization splits text into units. Stopword removal filters selected common words from a later analysis, such as keyword counts. PromptSage retains stopwords in the original prompt.

**2. Are words and tokens the same?**  
No. A word tokenizer creates word-like units, while BPE can split a word into multiple encoding-specific pieces. Punctuation and Unicode can also affect token boundaries.

**3. Is PromptSage a machine-learning model?**  
No. It uses classical NLP preprocessing, counters, regular expressions, and fixed deterministic rewrite rules. It does not train or call a generative model.

**4. How does the optimizer work?**  
It checks a small list of known verbose phrases, replaces or removes selected patterns, then normalizes whitespace. The remaining words and constraints are kept as written.

**5. Why can token count vary by encoding?**  
Each encoding has its own vocabulary and merge rules, so the same text can map to a different sequence of token IDs.

**6. Does the rewrite guarantee the same meaning?**  
No. The rules are conservative, but users should inspect the suggested text and verify all instructions and constraints.

**7. Why use stopwords for keyword analysis?**  
Common words can dominate frequency lists without describing the topic. Filtering them from that analysis makes content-bearing terms easier to see.

## Run tests

```powershell
python -m unittest discover -s tests -v
```
