# HR Candidate Profile Parser

An end-to-end **HR Candidate Profile Parser** built with **Hugging Face Transformers** and **LangChain Structured Output Parsing**.

The project demonstrates how an LLM can convert unstructured candidate information into a consistent JSON-style profile containing:

- `full_name`
- `email`
- `education`
- `skills`
- `experience`

The implementation is provided in two notebooks:

1. **Text-based Candidate Profile Parser**
2. **PDF-based Candidate Profile Parser**

Each notebook has its own README, while this root README provides the overall architecture, implementation comparison, observed results, and limitations.

---

## Project Structure

```text
HR-Candidate-Profile-Parser/
│
├── README.md
│
├── Tips-Hindawi-Hr-Candidate-Profile-Parser-Text_Task.ipynb
├── README_Text_Task.md
│
├── Tips-Hindawi-Hr-Candidate-Profile-Parser-PDF_Task.ipynb
└── README_PDF_Task.md
```

> The repository is intentionally kept simple: each notebook is accompanied by its dedicated README, and the root `README.md` documents the complete project.

---

## Project Objective

The goal is to build a lightweight NLP/LLM pipeline that takes candidate resume information and transforms it into structured data suitable for downstream HR applications.

Instead of manually extracting fields from a resume, the LLM is instructed to identify the required information and return it in a predefined structure.

The expected profile schema is:

```json
{
  "full_name": "string",
  "email": "string",
  "education": [
    {
      "degree": "string",
      "institution": "string",
      "year": "string"
    }
  ],
  "skills": [
    "string"
  ],
  "experience": [
    {
      "role": "string",
      "company": "string",
      "years": "string"
    }
  ]
}
```

---

# Architecture

The two notebooks share the same core extraction architecture:

```text
                 ┌──────────────────────┐
                 │      Resume Input    │
                 └──────────┬───────────┘
                            │
                 ┌──────────▼───────────┐
                 │  Prompt Construction │
                 │    PromptTemplate    │
                 └──────────┬───────────┘
                            │
                 ┌──────────▼───────────┐
                 │ Hugging Face LLM     │
                 │ Text Generation      │
                 └──────────┬───────────┘
                            │
                            ▼
                    Generated JSON
                            │
                 ┌──────────▼───────────┐
                 │ JSON Block Extraction│
                 └──────────┬───────────┘
                            │
                 ┌──────────▼───────────┐
                 │ StructuredOutputParser│
                 └──────────┬───────────┘
                            │
                            ▼
                 Structured Python Dict
                            │
                 ┌──────────▼───────────┐
                 │ Schema Validation    │
                 └──────────────────────┘
```

The major difference is how the input reaches the prompt.

### Text Notebook

```text
Raw Resume Text
      ↓
PromptTemplate
      ↓
Mistral-Nemo-Instruct-2407
      ↓
JSON Extraction
      ↓
StructuredOutputParser
      ↓
Validated Profile
```

### PDF Notebook

```text
PDF Resume
    ↓
PyPDFLoader
    ↓
Extracted PDF Text
    ↓
PromptTemplate
    ↓
Mistral-7B-Instruct-v0.2
    ↓
JSON Extraction
    ↓
StructuredOutputParser
    ↓
Validated Profile
```

---

# Technologies Used

| Technology | Purpose |
|---|---|
| Python | Main programming language |
| Hugging Face Transformers | Loading and running the LLMs |
| Mistral-Nemo-Instruct-2407 | LLM used in the text-based implementation |
| Mistral-7B-Instruct-v0.2 | LLM used in the PDF-based implementation |
| LangChain | Prompt and structured-output components |
| `StructuredOutputParser` | Enforcing the required output fields |
| `ResponseSchema` | Defining the candidate profile schema |
| `PromptTemplate` | Building the extraction prompt |
| PyPDF | PDF dependency |
| `PyPDFLoader` | Extracting text from PDF resumes |
| PyTorch | Model execution |

---

# Notebook 1 — Text-Based Parser

**Notebook:** `Tips-Hindawi-Hr-Candidate-Profile-Parser-Text_Task.ipynb`

This notebook starts with a controlled resume snippet supplied directly as a Python string.

### Input

The notebook uses a sample candidate profile containing:

- Full name
- Email
- Education
- Skills
- Two work experiences

### Model

```text
mistralai/Mistral-Nemo-Instruct-2407
```

### Main workflow

1. Install required packages.
2. Import Transformers and LangChain components.
3. Load the Mistral-Nemo model and tokenizer.
4. Define the text-generation function.
5. Define the five `ResponseSchema` objects.
6. Create a `StructuredOutputParser`.
7. Generate format instructions.
8. Build the resume extraction prompt.
9. Generate the LLM response.
10. Extract the JSON object from the model response.
11. Parse the JSON using `StructuredOutputParser`.
12. Display the extracted candidate profile.
13. Run assertions to validate the required fields and data types.

### Observed result

The notebook successfully extracted:

- The candidate's full name
- Email
- One education entry
- Three skills
- Two experience entries

The model returned the requested JSON structure, and the final validation cell completed successfully.

The sample result therefore demonstrates that the parser pipeline works correctly on clean, explicitly formatted resume text.

---

# Notebook 2 — PDF-Based Parser

**Notebook:** `Tips-Hindawi-Hr-Candidate-Profile-Parser-PDF_Task.ipynb`

The second notebook extends the idea to a real PDF resume.

### Model

```text
mistralai/Mistral-7B-Instruct-v0.2
```

### Input pipeline

The notebook uses:

```python
PyPDFLoader
```

to load the resume PDF and convert its pages into text.

The extracted page contents are then combined into one string:

```python
pdf_text = "\n\n".join(
    document.page_content
    for document in documents
)
```

This means the LLM does not directly process the PDF file. It processes the text extracted from the PDF.

### Main workflow

1. Install LangChain, Transformers, and `pypdf`.
2. Import `PyPDFLoader`.
3. Load Mistral-7B.
4. Define the generation function.
5. Define the same five candidate-profile schemas.
6. Create the `StructuredOutputParser`.
7. Load the PDF.
8. Extract and combine the PDF text.
9. Inspect the extracted text.
10. Build the extraction prompt.
11. Generate the structured response.
12. Extract the JSON block.
13. Parse it with `StructuredOutputParser`.
14. Display the extracted profile.
15. Validate the required fields and data types.

### Observed result

The notebook successfully extracted a much richer profile from the actual CV.

The result contained:

- Full name
- Email
- Education
- A large list of skills
- Multiple experience/project/learning-related entries

The final validation also completed successfully.

---

# Comparison

| Aspect | Text Notebook | PDF Notebook |
|---|---|---|
| Input | Direct text string | PDF resume |
| Input extraction | Not required | `PyPDFLoader` |
| Model | Mistral-Nemo-Instruct-2407 | Mistral-7B-Instruct-v0.2 |
| Prompting | HR extraction prompt | HR extraction prompt |
| Output format | Structured JSON | Structured JSON |
| Parser | `StructuredOutputParser` | `StructuredOutputParser` |
| JSON extraction | Yes | Yes |
| Validation | Assertions | Assertions |
| Validation result | Passed | Passed |
| Main purpose | Controlled proof of concept | Real resume processing |

---

# What Happened in the Results?

## 1. Both implementations completed the full pipeline

Both notebooks successfully followed the complete path:

```text
Input
→ Prompt
→ LLM
→ JSON
→ StructuredOutputParser
→ Validation
```

The validation cells confirmed that the required top-level fields were present and that the list-based fields had the expected Python data type.

---

## 2. The Text implementation is easier to control

The text notebook uses a short, clean input whose sections are clearly identified.

Because the input is controlled, the expected information is easy for the model to identify.

The output closely follows the provided information:

```text
John Smith
john.smith@email.com
B.Sc. Computer Science
MIT
Python
Machine Learning
Data Analysis
Software Engineer
Google
Data Scientist
OpenAI
```

This makes the notebook useful as a baseline and as a proof that the structured-output pipeline works.

---

## 3. The PDF implementation is closer to a real HR use case

The PDF notebook introduces an important real-world step:

```text
PDF → Text Extraction → LLM Parsing
```

This makes the solution more practical because resumes are commonly stored as PDF documents.

The actual CV produced a substantially larger profile, including many technical skills and several experience/project/education-related entries.

---

# Important Observation About the PDF Result

The PDF output is not simply a clean copy of a manually labeled resume.

The model interpreted different sections of the CV and placed information into the requested categories.

For example, the extracted `skills` list contains technical items such as:

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

It also contains broader labels such as:

```text
Frameworks
Tools
Languages
Data
```

Similarly, the `experience` list includes not only conventional employment information but also entries related to internships, learning programs, projects, and academic activities.

This is an important characteristic of LLM-based extraction: the model can normalize heterogeneous resume content, but the resulting categories still depend on the prompt and the structure of the original CV.

---

### What can be concluded from the current experiments?

- **Mistral-Nemo-Instruct-2407** successfully handled the controlled text extraction task.
- **Mistral-7B-Instruct-v0.2** successfully handled the PDF-to-text-to-structured-profile workflow.
- The PDF notebook demonstrates the more complete end-to-end application scenario.
- The current results do **not** constitute a controlled benchmark proving that one model has better extraction quality than the other.

---

# Output Parsing Strategy

The project does not simply trust the model's response.

The model may return JSON inside a Markdown code block:

```text
```json
{
    ...
}
```
```

The notebooks therefore use an `extract_json_block()` helper.

It first searches for fenced JSON:

```python
fenced_pattern = r'```json\s*(.*?)\s*```'
```

If no fenced block is found, it falls back to locating the first `{` and the last `}`.

After extracting the JSON text, the result is passed to:

```python
output_parser.parse(json_text)
```

This gives the project an additional structural validation layer.

---

# Why StructuredOutputParser Is Important

The LLM is responsible for understanding the resume and generating the candidate information.

`StructuredOutputParser` is responsible for checking whether the response follows the requested schema.

The architecture therefore separates:

```text
Information Extraction
        +
Output Structure Enforcement
```

This is useful for downstream applications because the result can be consumed as structured Python data instead of arbitrary natural-language text.

---

# Validation

Both notebooks use assertions such as:

```python
assert "full_name" in output_data
assert "email" in output_data
assert "education" in output_data
assert "skills" in output_data
assert "experience" in output_data
```

They also verify that:

```python
education
skills
experience
```

are lists.

A successful validation message indicates that the generated result satisfied the required top-level schema checks.

---

# Limitations

## 1. No ground-truth evaluation

The notebooks validate schema compliance, but they do not calculate extraction metrics such as:

- Precision
- Recall
- F1-score
- Field-level accuracy

Therefore, a successful parser validation does not automatically mean that every extracted value is factually correct.

---

## 2. PDF extraction depends on the PDF format

`PyPDFLoader` is suitable for text-based PDFs.

A scanned PDF containing only images would generally require an OCR stage before sending the text to the LLM.

That OCR stage is not part of the current implementation.

---

## 3. LLM output is probabilistic

The generation configuration uses sampling:

```python
do_sample=True
top_k=50
top_p=0.95
temperature=0.7
```

Consequently, repeated executions may not always produce byte-for-byte identical output.

---

## 4. Category interpretation can vary

A resume may contain:

- employment
- internships
- academic projects
- courses
- training programs
- volunteer work

The current schema has only:

```text
experience → role, company, years
```

Therefore, the model may place some academic or training activities inside `experience`.

A production HR system would likely benefit from a richer schema separating:

```text
employment
internships
projects
education
certifications
training
```

---

# Improvements That Could Be Added Later

Possible next steps include:

- Add OCR for scanned resumes.
- Introduce a standardized evaluation dataset.
- Run both models on the same CVs.
- Compare field-level extraction accuracy.
- Add confidence scores.
- Add certifications and projects as separate schema fields.
- Normalize dates into a consistent format.
- Normalize skill names.
- Detect duplicate skills.
- Add Pydantic-based validation.
- Build a batch-processing pipeline for multiple resumes.
- Store parsed candidates in a database.
- Add a human-review step for uncertain fields.

---

# Conclusion

This project demonstrates a practical LLM-based approach for converting unstructured resume information into structured candidate profiles.

The **Text notebook** establishes the core extraction and structured-output pipeline using controlled input.

The **PDF notebook** extends the same concept into a more realistic workflow by adding PDF text extraction before LLM processing.

Both implementations successfully completed their required validation checks.

The most important architectural idea is:

```text
Unstructured Resume
        ↓
Text Extraction
        ↓
LLM Information Extraction
        ↓
JSON
        ↓
StructuredOutputParser
        ↓
Validated Candidate Profile
```

The project provides a foundation that can be extended into a larger automated HR screening or candidate-profile processing system.
