# CI/CD Pipeline - Submission Document

**Group Members:** [Votre nom ici]

---

## 1. Public GitHub Repository URL

🔗 **Repository URL:** `https://github.com/LuzBoger/web-security`

---

## 2. Screenshot of CI Pipeline Passing

📸 **CI Pipeline Screenshot:** [To be added after first successful run]

**Location:** GitHub → Actions tab → CI workflow

The CI pipeline should show:
- ✅ Test job completed successfully (Python 3.8, 3.9, 3.10)
- ✅ Trivy scan job completed successfully
- ✅ All steps passed

---

## 3. Screenshot of CD Pipeline Passing

📸 **CD Pipeline Screenshot:** [To be added after first successful run]

**Location:** GitHub → Actions tab → CD workflow

The CD pipeline should show:
- ✅ Build job completed successfully
- ✅ Docker image built and pushed to Docker Hub

---

## 4. Screenshot of Docker Hub Repository

📸 **Docker Hub Screenshot:** [To be added after first successful deployment]

**Location:** Docker Hub → Repositories → arthurbrd/calculator

Should display:
- Repository name: `arthurbrd/calculator`
- Tag: `latest`
- Last pushed timestamp
- Image size

---

## 5. Created Files

### 5.1 CI/CD Workflow Files

#### `.github/workflows/ci.yml`

```yaml
name: CI

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  security-events: write
  actions: read

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.8', '3.9', '3.10']
    
    steps:
      - name: checkout
        uses: actions/checkout@v5
      
      - name: Python ${{ matrix.python-version }}
        uses: actions/setup-python@v6
        with:
          python-version: ${{ matrix.python-version }}
      
      - name: dependencies
        run: |
          python -m pip install --upgrade pip
          pip install flake8 pytest
          if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
      
      - name: flake8
        run: |
          # Stop the build if there are Python syntax errors or undefined names
          flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
          # Exit-zero treats all errors as warnings
          flake8 . --count --exit-zero --statistics
      
      - name: pytest
        run: |
          pytest calculator/tests/

  trivy-scan:
    runs-on: ubuntu-latest
    
    steps:
      - name: checkout
        uses: actions/checkout@v5
      
      - name: trivy FS mode
        uses: aquasecurity/trivy-action@0.33.1
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'results.sarif'
          severity: 'CRITICAL,HIGH'
      
      - name: upload
        uses: github/codeql-action/upload-sarif@v4
        with:
          sarif_file: 'results.sarif'
```

#### `.github/workflows/cd.yml`

```yaml
name: CD

on:
  workflow_run:
    workflows: ["CI"]
    types:
      - completed
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    
    steps:
      - name: checkout
        uses: actions/checkout@v5
      
      - name: login
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: build and push
        uses: docker/build-push-action@v6
        id: push
        with:
          context: .
          file: ./calculator/Dockerfile
          push: true
          tags: arthurbrd/calculator:latest
```

---

### 5.2 Application Files (Already Provided)

#### `calculator/calculator.py`

```python
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

if __name__ == "__main__":
    print(add(1, 2))
    print(subtract(1, 2))
    print(multiply(1, 2))
    print(divide(1, 2))
```

#### `calculator/tests/test_calculator.py`

```python
import unittest

from calculator.calculator import add, divide, multiply, subtract


class TestCalculator(unittest.TestCase):
    def test_add(self):
        self.assertEqual(add(1, 2), 3)
        self.assertEqual(add(-1, 1), 0)
        self.assertEqual(add(-1, -1), -2)

    def test_subtract(self):
        self.assertEqual(subtract(1, 2), -1)
        self.assertEqual(subtract(-1, 1), -2)
        self.assertEqual(subtract(-1, -1), 0)

    def test_multiply(self):
        self.assertEqual(multiply(1, 2), 2)
        self.assertEqual(multiply(-1, 1), -1)
        self.assertEqual(multiply(-1, -1), 1)

    def test_divide(self):
        self.assertEqual(divide(1, 2), 0.5)
        self.assertEqual(divide(-1, 1), -1)
        self.assertEqual(divide(-1, -1), 1)
        self.assertEqual(divide(0, 5), 0)
        with self.assertRaises(ValueError):
            divide(1, 0)

if __name__ == '__main__':
    unittest.main()
```

#### `calculator/Dockerfile`

```dockerfile
FROM python:3.10-alpine

COPY . /app

WORKDIR /app

CMD ["python", "calculator/calculator.py"]
```

#### `requirements.txt`

```
pytest==7.4.0
flake8==6.0.0
```

---

## 6. Troubleshooting CD Pipeline

Si le workflow CD ne se déclenche pas automatiquement :

### Solution appliquée :
Ajout du filtre `branches: [main]` dans le trigger `workflow_run` pour s'assurer que le CD ne se déclenche que pour les exécutions CI sur la branche main.

### Vérifications à faire :
1. **Vérifier que le workflow CI s'appelle bien "CI"** (nom exact dans le fichier ci.yml)
2. **Vérifier les permissions** : Le workflow CD doit avoir les permissions pour lire les événements de workflow
3. **Vérifier les secrets** : `DOCKER_USERNAME` et `DOCKER_PASSWORD` doivent être configurés
4. **Déclencher manuellement** : Vous pouvez aussi déclencher le CD manuellement en ajoutant `workflow_dispatch` au trigger

### Alternative : Déclencher manuellement le CD
Si le problème persiste, vous pouvez déclencher le CD manuellement depuis l'onglet Actions de GitHub.

---

## 7. Next Steps

1. ✅ CI/CD workflows created
2. ✅ Code pushed to GitHub repository
3. ✅ CI pipeline verified passing
4. ⏳ Verify CD pipeline triggers after CI success
5. ⏳ Check Docker Hub for pushed image
6. ⏳ Take screenshots for submission
7. ⏳ Update this document with screenshots

---

## 8. Resources Used

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Pytest Documentation](https://docs.pytest.org/)
- [Flake8 Documentation](https://flake8.pycqa.org/)
- [Trivy GitHub Action](https://github.com/aquasecurity/trivy-action)
- [Docker Build Push Action](https://github.com/docker/build-push-action)
- [Workflow Run Event](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#workflow_run)

---

**Date:** 2025-12-01  
**Project:** Calculator CI/CD Pipeline  
**Docker Hub:** arthurbrd/calculator  
**GitHub Repository:** https://github.com/LuzBoger/web-security
