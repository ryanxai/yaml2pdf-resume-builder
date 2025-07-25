# CV Generator

![Resume Preview](docs/resume_preview_top_half.png)

## Overview

This project allows you to maintain your resume content in an easy-to-edit JSON format while generating a professionally styled PDF using Python and LaTeX. You can run it either as a **command-line tool**, a **FastAPI web service**, or with **Docker**.

## Usage Options

### Option 1: FastAPI Web Service (NEW! 🚀)

> **🚀 Live Demo**: [https://cvgen-c-jysq.fly.dev](https://cvgen-c-jysq.fly.dev)
> 
> **📖 API Documentation**: [https://cvgen-c-jysq.fly.dev/docs](https://cvgen-c-jysq.fly.dev/docs)

Run the resume builder as a web API service that accepts HTTP requests.

#### Requirements
- Python 3.6+
- All dependencies from `requirements.txt`
- LaTeX distribution (MacTeX, MiKTeX, or TeXLive)
- OpenRouter API key (for LLM features)

#### Environment Variables
Copy `env.example` to `.env` and add your OpenRouter API key:
```bash
cp env.example .env
# Edit .env and replace with your actual API key
```

You can get a free API key from [OpenRouter](https://openrouter.ai/).

#### Quick Start

```bash
# Install dependencies
pip install -r requirements.txt

# Start the FastAPI server
python3 main.py

# Or use uvicorn directly for development
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

The API will be available at `http://localhost:8000`

#### API Endpoints

- **GET `/`** - API information and available endpoints
- **GET `/health`** - Health check endpoint
- **POST `/generate-resume`** - Generate PDF from JSON resume data
- **POST `/upload-json`** - Upload a JSON file and generate PDF
- **POST `/generate-from-json`** - Generate resume.tex and resume.pdf from existing resume.json
- **POST `/improve-resume-section`** - Improve any resume section using LLMs
- **GET `/download/{filename}`** - Download generated PDF files
- **GET `/sample-data`** - Get sample resume data structure
- **GET `/template`** - Get the LaTeX template content
- **DELETE `/cleanup`** - Clean up temporary files (dev endpoint)

#### API Documentation

FastAPI automatically generates interactive API documentation:
- **Swagger UI**: `http://localhost:8000/docs`
- **ReDoc**: `http://localhost:8000/redoc`

#### Usage Examples

**Generate resume with JSON data:**
```bash
curl -X POST "http://localhost:8000/generate-resume" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "contact": {
      "phone": "+1 (555) 123-4567",
      "email": "john.doe@example.com",
      "location": "Boston, MA",
      "links": [
        {"name": "GitHub", "url": "https://github.com/johndoe"},
        {"name": "LinkedIn", "url": "https://linkedin.com/in/johndoe"}
      ]
    },
    "summary": "Experienced developer...",
    "skills": [
      {"category": "Programming", "items": "Python, JavaScript"}
    ],
    "experience": [...],
    "education": [...],
    "awards": [],
    "certifications": [],
    "publications": []
  }'
```

**Upload JSON file:**
```bash
curl -X POST "http://localhost:8000/upload-json" \
  -F "file=@resume.json"
```

**Generate from existing resume.json:**
```bash
curl -X POST "http://localhost:8000/generate-from-json"
```

**Improve any resume section using LLMs:**
```bash
curl -X POST "http://localhost:8000/improve-resume-section" \
  -H "Content-Type: application/json" \
  -d '{
    "instructions": "You are a professional recruiter. Improve the writing style to be more professional and concise. Don't provide any other text than the improved section.",
    "section_name": "Summary",
    "section_text": "Senior Data Scientist and ML Engineer with 10+ years of expertise in NLP, speech processing, and generative AI. Demonstrated success developing LLM-based systems achieving high accuracy metrics. Specialized in building end-to-end AI solutions from concept to production, combining deep learning architectures with practical business applications for start-ups and big companies."
  }'
```

**Improve experience section:**
```bash
curl -X POST "http://localhost:8000/improve-resume-section" \
  -H "Content-Type: application/json" \
  -d '{
    "instructions": "Add more quantifiable achievements and make the language more impactful.",
    "section_name": "Experience",
    "section_text": "Led development team of 5 engineers to build a new product feature that increased user engagement by 25%."
  }'
```

**Download generated PDF:**
```bash
curl "http://localhost:8000/download/resume.pdf" \
  --output resume.pdf
```

**Test the API:**
```bash
# Run the provided test script
python3 test_api.py

# Test the improve-resume-section endpoint
python3 test_improve_summary.py

# Run examples of improve-resume-section endpoint
python3 example_improve_summary.py
```

### Option 2: Command Line (Original)

#### Requirements
- Python 3.6+
- LaTeX distribution (MacTeX, MiKTeX, or TeXLive)

#### Quick Start

```bash
# Activate the virtual environment (if you have one)
source venv/bin/activate

# Generate your resume
python generate_resume.py
```

### Option 3: Docker

#### Requirements
- Docker

#### For FastAPI Service

```bash
# Build the container
docker build -t resume-generator .

# Run the FastAPI server with volume mounting (recommended)
docker run -p 8000:8000 -v $(pwd):/app resume-generator

# Access the API at http://localhost:8000
```

#### For Command Line (Original)

```bash
# Build the container
docker build -t resume-generator .

# Run the original command-line version
docker run --rm -v $(pwd):/app resume-generator python3 generate_resume.py
```

#### Important Note About Docker Volume Mounting

When using the `/generate-from-json` endpoint, you **must** use volume mounting (`-v $(pwd):/app`) to make the generated `resume.tex` and `resume.pdf` files accessible in your host project directory. Without volume mounting, the files will be generated inside the container but won't be accessible from your host system.

## How It Works

1. Your resume content is stored in `resume.json` as structured JSON data
2. The LaTeX template (`template.tex`) defines the styling and layout
3. The Python script (`generate_resume.py`) combines your data with the template
4. The FastAPI service (`main.py`) provides web endpoints for the same functionality
5. The endpoints can generate files either with personalized names or as standard `resume.tex` and `resume.pdf`
   - `/generate-resume` and `/upload-json`: Generate files with personalized names
   - `/generate-from-json`: Generate standard `resume.tex` and `resume.pdf` from existing `resume.json`

## Creating Your Resume

1. **Edit Your Resume Data**:
   - Update `resume.json` with your personal information
   - Follow the existing structure for experience, education, etc.

2. **Customizing the Template** (Optional):
   - Edit `template.tex` to change the styling and layout

## API Integration

The FastAPI service can be easily integrated into other applications:

### Python Client Example
```python
import requests

# Generate resume
response = requests.post('http://localhost:8000/generate-resume', json=resume_data)
result = response.json()

# Download PDF
pdf_response = requests.get(f"http://localhost:8000{result['download_url']}")
with open('resume.pdf', 'wb') as f:
    f.write(pdf_response.content)

# Improve any resume section using LLMs
improve_data = {
    "instructions": "You are a professional recruiter. Improve the writing style to be more professional and concise.",
    "section_name": "Summary",
    "section_text": "Your current section text here..."
}
improve_response = requests.post('http://localhost:8000/improve-resume-section', json=improve_data)
improved_section = improve_response.json()['improved_section']
```

### JavaScript/Node.js Client Example
```javascript
const FormData = require('form-data');
const fs = require('fs');

// Upload JSON file
const form = new FormData();
form.append('file', fs.createReadStream('resume.json'));

fetch('http://localhost:8000/upload-json', {
  method: 'POST',
  body: form
})
.then(response => response.json())
.then(data => console.log(data));
```

## Deployment

### Automatic Deployment with GitHub Actions

This project includes automatic deployment to Fly.io using GitHub Actions. The deployment automatically manages secrets and performs health checks.

#### Quick Setup

1. **Run the setup script**:
   ```bash
   ./scripts/setup-deployment.sh
   ```

2. **Add GitHub Secrets**:
   - Go to your GitHub repository → Settings → Secrets and variables → Actions
   - Add `FLY_API_TOKEN` (from the setup script)
   - Add `OPENROUTER_API_KEY` (your OpenRouter API key)

3. **Deploy**:
   - Push to the `main` branch to trigger automatic deployment
   - Or manually trigger from GitHub Actions tab

#### Manual Deployment

```bash
# Set secrets
flyctl secrets set OPENROUTER_API_KEY=your_api_key_here

# Deploy
flyctl deploy
```

For detailed deployment instructions, see [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md).

## Development
```bash
# Install dependencies
pip install -r requirements.txt

# Run with auto-reload
uvicorn main:app --reload

# Run tests
python3 test_api.py
```

### Project Structure
```
cvgen/
├── main.py                    # FastAPI application
├── generate_resume.py         # Core resume generation logic
├── test_api.py               # API test script
├── test_improve_summary.py   # Test script for improve-resume-section endpoint
├── example_improve_summary.py # Examples of improve-resume-section usage
├── template.tex              # LaTeX template
├── resume.json               # Sample resume data
├── requirements.txt          # Python dependencies
├── env.example               # Environment variables example
├── Dockerfile               # Docker configuration
└── README.md                # This file
```
