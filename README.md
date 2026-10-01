# Resume Parser & Job Matcher

An AI-powered tool that parses resumes (PDF/DOCX), extracts structured candidate data using an LLM, and scores each candidate against a given job description — ranking the best and worst matches automatically.

## Features

- **Job Description Parsing** — Extracts role, required/preferred skills, minimum experience, education requirements, and responsibilities from a raw job description into a structured schema.
- **Resume Parsing** — Reads `.pdf` and `.docx` resumes and extracts candidate details (name, contact info, skills, experience, education, projects, certifications) regardless of section heading style (e.g. "Work History" vs "Experience" vs "Internships").
- **AI-Powered Matching** — Compares each parsed resume against the job description and returns a match score (0–100%), matching/missing skills, experience fit, and a short verdict.
- **Batch Processing** — Processes every resume in a folder, then ranks and prints the **top 2** and **bottom 2** candidates.

## Tech Stack

- **[Groq API](https://console.groq.com/)** — LLM inference (`openai/gpt-oss-120b`)
- **[Pydantic](https://docs.pydantic.dev/)** — Structured output validation via JSON schema
- **[pypdf](https://pypi.org/project/pypdf/)** — PDF text extraction
- **[python-docx](https://pypi.org/project/python-docx/)** — DOCX text extraction
- **[python-dotenv](https://pypi.org/project/python-dotenv/)** — Environment variable management

## Setup

### 1. Clone the repository
```bash
git clone git@github.com:shaloo22/Resume_parser.git
cd Resume_parser
```

### 2. Create a virtual environment and install dependencies
Using `uv`:
```bash
uv venv
uv add groq pydantic pypdf python-docx python-dotenv
```

Or using `pip`:
```bash
python -m venv .venv
.venv\Scripts\activate      # Windows
pip install groq pydantic pypdf python-docx python-dotenv
```

### 3. Set up your API key
Create a `.env` file in the project root:
```
GROQ_API_KEY=your_groq_api_key_here
```

> **Note:** `.env` is already in `.gitignore` — never commit your API key.

### 4. Add resumes
Place candidate resumes (`.pdf` or `.docx`) inside a `resumes/` folder in the project root.

## Usage

Edit the `job_description` variable in the script with the job posting you want to match candidates against, then run:

```bash
uv run python resume_parser.py
```

Or:
```bash
python resume_parser.py
```

### Sample Output
```
Processing: john_doe_resume.pdf
Score: 82.5

Processing: jane_smith_resume.pdf
Score: 91.0

TOP 2 CANDIDATES
Jane Smith - 91.0 %
{...match details...}
John Doe - 82.5 %
{...match details...}

LOWEST 2 CANDIDATES
...
```

## How It Works

1. **Job description** is sent to the LLM with a Pydantic-defined schema (`JobDescription`) → returns structured JSON (role, skills, experience, etc.).
2. **Each resume** is read (PDF/DOCX → plain text), then sent to the LLM with a `Resume` schema → returns structured candidate data.
3. **Job + Resume** are compared in a second LLM call, returning a `MatchResult` (score + details) based on skill overlap, experience match, and overall fit.
4. Results are sorted by score, and the **top 2** and **bottom 2** candidates are printed.

## Project Structure
```
Resume_parser/
├── resume_parser.py
├── resumes/              # Place candidate resumes here (not committed)
├── .env                  # Your API key (not committed)
├── .gitignore
└── README.md
```

## Notes

- A short delay (`time.sleep(5)`) is added between LLM calls to avoid hitting Groq's rate limits.
- The script currently uses `openai/gpt-oss-120b` — swap the `model` variable if your Groq account has access to a different model.
- Missing information in resumes/job descriptions returns `null` or an empty list rather than being invented by the LLM.
