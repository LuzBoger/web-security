# Calculator CI/CD Pipeline

A simple Python calculator application with a complete CI/CD pipeline using GitHub Actions.

## 🎯 Project Overview

This project implements a complete CI/CD pipeline for a Python calculator application that includes:
- Automated testing across multiple Python versions (3.8, 3.9, 3.10)
- Security vulnerability scanning with Trivy
- Automated Docker image building and deployment to Docker Hub

## 📋 Application Features

The calculator application provides basic arithmetic operations:
- Addition
- Subtraction
- Multiplication
- Division (with zero-division protection)

## 🚀 Local Setup

1. Clone the repository:
```bash
git clone <your-repo-url>
cd web-security
```

2. Run the calculator:
```bash
python calculator/calculator.py
```

## 🧪 Running Tests

```bash
pytest calculator/tests/
```

Or using unittest directly:
```bash
python -m unittest calculator/tests/test_calculator.py
```

## 🔄 CI/CD Pipeline

### Continuous Integration (CI)

The CI pipeline (`.github/workflows/ci.yml`) runs automatically on every push to `main` or `master` branch and includes:

**Test Job:**
- Matrix testing across Python 3.8, 3.9, and 3.10
- Code quality checks with flake8
- Automated test execution with pytest

**Trivy Scan Job:**
- Filesystem vulnerability scanning
- SARIF report generation for critical and high severity issues
- Results uploaded to GitHub Security tab

### Continuous Deployment (CD)

The CD pipeline (`.github/workflows/cd.yml`) triggers automatically after successful CI completion:
- Authenticates with Docker Hub
- Builds Docker image from `calculator/Dockerfile`
- Pushes image to Docker Hub as `arthurbrd/calculator:latest`

## 🔐 GitHub Secrets Configuration

Configure the following secrets in your GitHub repository (Settings → Secrets and variables → Actions):

- `DOCKER_USERNAME`: Your Docker Hub username
- `DOCKER_PASSWORD`: Your Docker Hub personal access token (with read/write permissions)

## 🐳 Docker

Build the Docker image locally:
```bash
docker build -t calculator -f calculator/Dockerfile .
```

Run the container:
```bash
docker run calculator
```

Pull from Docker Hub:
```bash
docker pull arthurbrd/calculator:latest
```

## � Project Structure

```
web-security/
├── calculator/
│   ├── __init__.py
│   ├── calculator.py          # Main calculator module
│   ├── Dockerfile             # Docker configuration
│   └── tests/
│       ├── __init__.py
│       └── test_calculator.py # Unit tests
├── .github/
│   └── workflows/
│       ├── ci.yml             # CI pipeline
│       └── cd.yml             # CD pipeline
├── requirements.txt           # Python dependencies
└── README.md
```

## 📝 License

This project is for educational purposes only.