# HR Candidate Profile Parser — PDF Task

This notebook implements the **PDF-based version** of the HR Candidate Profile Parser.

It extends the text-based implementation by introducing PDF document loading and text extraction before sending the resume to the LLM.

---

## Objective

Take a real PDF resume, extract its text, and convert the content into a structured candidate profile containing:

- `full_name`
- `email`
- `education`
- `skills`
- `experience`

---

# Pipeline

```text
PDF Resume
    ↓
PyPDFLoader
    ↓
Extracted Text
    ↓
PromptTemplate
    ↓
Mistral-7B-Instruct-v0.2
    ↓
Generated JSON
    ↓
JSON Block Extraction
    ↓
StructuredOutputParser
    ↓
Validated Candidate Profile
```

---

# Model

The notebook uses:

```text
mistralai/Mistral-7B-Instruct-v0.2
```

The model is loaded through Hugging Face Transformers.

It uses:

```python
torch_dtype=torch.float16
device_map="auto"
```

---

# PDF Processing

The notebook uses LangChain's:

```python
PyPDFLoader
```

The PDF is loaded with:

```python
loader = PyPDFLoader(pdf_path)
documents = loader.load()
```

The loader returns document objects representing the PDF pages.

The notebook also prints the number of pages to verify that the document was loaded.

---

# Combining PDF Text

The page contents are combined into one string:

```python
pdf_text = "\n\n".join(
    document.page_content
    for document in documents
)
```

This is an important architectural point:

> The LLM does not directly process the PDF file. The PDF is first converted into text, and that text becomes the LLM input.

The notebook prints the extracted text and also previews the first 5000 characters.

---

# Structured Output Schema

The same five candidate-profile fields are used:

```text
full_name
email
education
skills
experience
```

### Education

Each education item contains:

```text
degree
institution
year
```

### Experience

Each experience item contains:

```text
role
company
years
```

### Skills

A list of strings.

---

# Stronger Prompt for Real Resume Input

The PDF notebook uses a more explicit prompt than the basic text version.

It tells the model:

- All five fields are mandatory.
- Fields must never be omitted.
- Missing strings should become `""`.
- Missing list values should become `[]`.
- The model must not invent information.
- Education and experience must use the required nested fields.

This is particularly useful for real resumes because their structure is less predictable than the controlled sample text.

---

# Generation

The notebook uses:

```python
generate_text(
    prompt,
    max_new_tokens=3000
)
```

The larger generation budget is appropriate for a full CV because a real resume may contain significantly more information than the small sample used in the text notebook.

The same sampling configuration is used:

```python
do_sample=True
top_k=50
top_p=0.95
temperature=0.7
```

---

# Observed Output

The executed notebook successfully generated a structured profile for the actual CV.

The result included:

## Education

The model extracted:

```text
B.Sc.
King Salman International University (KSIU)
2021 – 2025
```

## Skills

The model extracted a large list of technical and general skills, including examples such as:

```text
Python
C++
Machine Learning
Deep Learning
Computer Vision
OpenCV
Generative AI
LLMs
RAG
Embeddings
PyTorch
TensorFlow
Scikit-learn
Git
GitHub
```

It also included broader labels such as:

```text
Frameworks
Tools
Languages
Data
```

This shows that the model interpreted the organization of the CV rather than simply copying one manually defined list.

## Experience / Activities

The extracted list included entries such as:

```text
Artificial Intelligence Intern — Uneeq Interns
AI Roadmap Mentor Chatbot Developer
AI Level 2 Student — Digital Hub (D-Hub) & ERI
AI Diploma Student — NeuroTech
Smart Landmine Detection Robot Developer — King Salman International University
```

Some entries did not have a company or year, so the structured output used empty strings for those values, consistent with the prompt.

---

# JSON Extraction

The notebook uses the same robust helper:

```python
extract_json_block()
```

It first searches for fenced JSON and then falls back to locating the JSON object between `{` and `}`.

This makes the parser less dependent on the exact Markdown formatting produced by the LLM.

---

# Structured Parsing

After extracting the JSON text:

```python
output_data = output_parser.parse(json_text)
```

LangChain validates the response against the five required schemas.

The parsed result is stored as a Python dictionary.

---

# Validation

The final validation checks:

```python
required_fields = [
    "full_name",
    "email",
    "education",
    "skills",
    "experience"
]
```

It then verifies that all fields exist and that:

```text
education
skills
experience
```

are lists.

The executed notebook ended with:

```text
PDF HR Candidate Profile Parser test passed successfully!
```

---

# Result

The PDF implementation successfully demonstrates the complete real-document workflow:

```text
Real PDF
→ PDF Text Extraction
→ LLM Information Extraction
→ JSON
→ StructuredOutputParser
→ Validation
```

---

# Strengths

- Works with a real resume PDF.
- Adds document ingestion to the LLM pipeline.
- Uses the same structured schema as the text implementation.
- Handles missing information with empty values/lists.
- Successfully passed the final schema validation.
- Produces a rich candidate profile from the actual CV.

---

# Important Observations

The PDF result also demonstrates an important limitation of general LLM extraction.

The model may classify training, academic activities, projects, and internships under the broad `experience` field.

Similarly, headings such as:

```text
Frameworks
Tools
Languages
```

may appear inside the `skills` list.

This is not necessarily a parser failure: it reflects the fact that the current schema is relatively broad and the model is interpreting a heterogeneous CV.

A production system could improve this by introducing separate fields such as:

```text
employment
internships
projects
training
certifications
```

and by defining stricter normalization rules.

---

# Limitations

## PDF Type

`PyPDFLoader` is designed for extracting text from PDFs.

If the resume is a scanned/image-only PDF, an OCR step would be needed before LLM extraction.

## Evaluation

The notebook checks schema validity, but it does not compare every extracted value against manually labeled ground truth.

Therefore:

```text
Schema validation ≠ extraction accuracy
```

## Model Comparison

This notebook uses a different model from the text notebook.

Because the input also changes from controlled text to a real PDF, the two notebooks cannot be used as a controlled benchmark for determining which model is better.

---

# Relationship to the Text Notebook

The text notebook establishes:

```text
Text → LLM → Structured JSON
```

This notebook extends it to:

```text
PDF → Text → LLM → Structured JSON
```

The core LangChain structured-output architecture remains the same.

The major addition is:

```python
PyPDFLoader
```

---

# Final Takeaway

The PDF notebook demonstrates that the candidate-profile parser can move beyond a toy text example and process a real resume document.

The complete architecture is:

```text
                PDF Resume
                    │
                    ▼
              PyPDFLoader
                    │
                    ▼
              Extracted Text
                    │
                    ▼
             PromptTemplate
                    │
                    ▼
          Mistral-7B-Instruct
                    │
                    ▼
              Generated JSON
                    │
                    ▼
          JSON Block Extraction
                    │
                    ▼
        StructuredOutputParser
                    │
                    ▼
          Validated Candidate
```

The result is a practical foundation for a larger automated HR candidate-processing system.
