# HR Candidate Profile Parser — Text Task

This notebook implements the **text-based version** of the HR Candidate Profile Parser.

The purpose is to demonstrate the core LLM extraction pipeline using a controlled resume text input before moving to a real PDF resume.

---

## Objective

Convert unstructured candidate text into a structured JSON-style profile containing:

- `full_name`
- `email`
- `education`
- `skills`
- `experience`

The expected structure is:

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
  "skills": ["string"],
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

# Pipeline

```text
Resume Text
     ↓
PromptTemplate
     ↓
Mistral-Nemo-Instruct-2407
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
mistralai/Mistral-Nemo-Instruct-2407
```

The model is loaded through Hugging Face Transformers:

```python
AutoTokenizer
AutoModelForCausalLM
```

The model is configured with:

```python
torch_dtype=torch.float16
device_map="auto"
```

---

# Main Components

## 1. Package Installation

The notebook installs the required LangChain and Transformers packages.

```python
!pip install -U langchain langchain-community langchain-core langchain-classic transformers==4.52.4
```

---

## 2. Imports

The implementation uses:

- Hugging Face Transformers for the LLM
- LangChain `StructuredOutputParser`
- LangChain `ResponseSchema`
- LangChain `PromptTemplate`
- PyTorch
- Regular expressions for JSON extraction

---

## 3. Model Loading

The tokenizer and language model are loaded from Hugging Face.

The notebook uses the model as a text-generation engine rather than relying on a hosted API.

---

## 4. Text Generation Function

The helper function:

```python
generate_text(prompt, max_new_tokens=600)
```

takes the formatted prompt, tokenizes it, generates the completion, and decodes only the newly generated tokens.

The generation settings are:

```python
do_sample=True
top_k=50
top_p=0.95
temperature=0.7
```

These parameters allow controlled but non-deterministic generation.

---

# Structured Output Schema

Five `ResponseSchema` objects define the expected candidate profile.

### Full Name

```text
full_name → candidate's full name
```

### Email

```text
email → candidate's email address
```

### Education

A list of objects containing:

```text
degree
institution
year
```

### Skills

A list of strings.

### Experience

A list of objects containing:

```text
role
company
years
```

These schemas are combined into:

```python
StructuredOutputParser.from_response_schemas(response_schemas)
```

---

# Format Instructions

The parser generates formatting instructions using:

```python
output_parser.get_format_instructions()
```

Those instructions are inserted into the prompt.

This creates a connection between the expected schema and the LLM instruction.

---

# Input

The notebook uses a controlled sample:

```text
John Smith – john.smith@email.com

Education:
B.Sc. Computer Science, MIT, 2020

Skills:
Python, Machine Learning, Data Analysis

Experience:
- Software Engineer at Google (2020–2023)
- Data Scientist at OpenAI (2023–Present)
```

This input intentionally contains all required categories.

---

# Prompt Design

The prompt instructs the model to:

1. Act as an HR candidate profile parser.
2. Extract the candidate information.
3. Return only JSON.
4. Include all five required fields.
5. Use the required nested structures for education and experience.
6. Avoid inventing information.
7. Return empty values/lists when information is unavailable.

The parser's format instructions are inserted dynamically.

---

# Model Output

The executed notebook produced a JSON response containing:

- `John Smith`
- `john.smith@email.com`
- `B.Sc. Computer Science` from `MIT`
- `Python`
- `Machine Learning`
- `Data Analysis`
- Software Engineer at Google
- Data Scientist at OpenAI

The generated structure matched the required candidate-profile schema.

---

# JSON Extraction

The notebook uses:

```python
extract_json_block()
```

The helper supports two cases:

### JSON inside a Markdown code block

```text
```json
{
    ...
}
```
```

### Raw JSON

```text
{
    ...
}
```

The extracted JSON is then passed to the LangChain parser.

---

# Structured Parsing

The model response is converted into structured Python data using:

```python
output_data = output_parser.parse(json_text)
```

The result is a Python dictionary containing the candidate profile.

This is important because the application no longer has to manually interpret arbitrary model text.

---

# Validation

The final notebook cell checks that all required fields exist:

```python
assert "full_name" in output_data
assert "email" in output_data
assert "education" in output_data
assert "skills" in output_data
assert "experience" in output_data
```

It also checks that:

```text
education
skills
experience
```

are lists.

The executed notebook ended with:

```text
HR Candidate Profile Parser test passed successfully!
```

---

# Result

The text implementation successfully proves the basic pipeline:

```text
Controlled Text
→ LLM
→ JSON
→ StructuredOutputParser
→ Validation
```

This makes it a useful baseline before introducing PDF processing.

---

# Strengths

- Simple input pipeline.
- Easy to debug.
- Clear schema.
- Controlled test data.
- Successful structured parsing.
- Successful validation.
- Good baseline for the PDF implementation.

---

# Limitations

The input is manually supplied as a Python string, so this notebook does not yet solve the document-ingestion problem.

It also does not measure extraction accuracy using a ground-truth dataset.

The model uses sampling, so repeated runs may produce slightly different outputs.
