# cloud-security-flask-app-

(AI was used in assistance with troubleshooting the scripts and code) 

Overview
A guide to building a Dockerized Flask application and scanning it for vulnerabilities. This project demonstrates container creation  and security scanning.

Table of Contents:

Project Setup

Security Scanning

Security Verification

------------------------------------------------------------------------------
1. PROJECT SET UP
   
-  step 1: Create Project Structure
   mkdir secure-docker-app
   cd secure-docker-app
------------------------------------------------------------------------
- step 2: Create Flask App
  
  from flask import Flask, jsonify
import os
import datetime

app = Flask(__name__)

@app.route('/')
def hello():
    return jsonify({
        'message': 'Hello from secure Docker app!',
        'environment': os.getenv('ENVIRONMENT', 'development'),
        'server_time': datetime.datetime.now().isoformat(),
        'security_note': 'This app runs with non-root user in container'
    })

@app.route('/health')
def health():
    return jsonify({'status': 'healthy'})

@app.route('/info')
def info():
    return jsonify({
        'python_version': os.sys.version,
        'user': os.getenv('USER', 'unknown'),
        'container_id': os.getenv('HOSTNAME', 'unknown')
    })

if __name__ == '__main__':
    port = int(os.getenv('PORT', 5000))
    app.run(host='0.0.0.0', port=port, debug=False)
    --------------------------------------------------------------------------
- Step 3: Create Requirements File
  Flask==2.3.3
  Werkzeug==2.3.7
--------------------------------------------------------------------------
- step 4: Create secure Dockerfile
  
FROM python:3.9-slim-bullseye

# Security environment variables
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV PIP_NO_CACHE_DIR=1
ENV FLASK_ENV=production

# Create non-root user with specific UID
RUN groupadd -r appgroup && useradd -r -u 1001 -g appgroup appuser

WORKDIR /app

# Install only necessary system packages
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements and install dependencies
RUN pip install --no-cache-dir --upgrade pip && \
pip install --no-cache-dir -r requirements.txt

# Change ownership to non-root user
RUN chown -R appuser:appgroup /app

# Switch to non-root user
USER appuser

EXPOSE 5000

# Health check for container monitoring
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:5000/health || exit 1

CMD ["python", "app.py"]

------------------------------------------------------------------------------------------
- step 5: Create .dockerignore
  
  # Exclude sensitive and unnecessary files
__pycache__/
*.py[cod]
.env
.git/
.gitignore
*.log
Dockerfile
.dockerignore

- step 6: create a yaml file
  
  version: '3.8'
services:
  web:
    build: .
    ports:
      - "5000:5000"
    environment:
      - ENVIRONMENT=production
      - FLASK_DEBUG=False
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
  
----------------------------------------------------------------------------------------------------------------
SECURITY SCANNING

- step 1: build docker image and then scan it for vulnerabilities
  docker build -t secure-flask-app (TO BUILD) (ALSO ENSURE YOU RUN HEALTHCHECKS ON THE APP)
it shoud look something like this

<img width="958" height="997" alt="image" src="https://github.com/user-attachments/assets/7f9ee922-04c4-4e77-913b-adb4f41c3f83" />
-------------------------------------------------------------------------------------------------------------------------------------

  docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy:latest image secure-flask-app --severity HIGH,CRITICAL (VULNERABILITY SCAN)

  your out put should be something like this but alot longer

  C:\Users\cross\secure-docker-app>docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image secure-flask-app

2025-09-13T21:07:36Z    INFO    [vulndb] Need to update DB
2025-09-13T21:07:36Z    INFO    [vulndb] Downloading vulnerability DB...
2025-09-13T21:07:36Z    INFO    [vulndb] Downloading artifact...        repo="mirror.gcr.io/aquasec/trivy-db:2"
5.03 MiB / 70.40 MiB [---->__________________________________________________________] 7.15% ? p/s ?13.20 MiB / 70.40 MiB [----------->_________________________________________________] 18.75% ? p/s ?21.56 MiB / 70.40 MiB [------------------>__________________________________________] 30.63% ? p/s ?29.56 MiB / 70.40 MiB [-------------------->___________________________] 41.99% 40.88 MiB p/s ETA 0s38.03 MiB / 70.40 MiB [------------------------->______________________] 54.02% 40.88 MiB p/s ETA 0s46.22 MiB / 70.40 MiB [------------------------------->________________] 65.65% 40.88 MiB p/s ETA 0s54.70 MiB / 70.40 MiB [------------------------------------->__________] 77.69% 40.95 MiB p/s ETA 0s63.02 MiB / 70.40 MiB [------------------------------------------>_____] 89.53% 40.95 MiB p/s ETA 0s70.40 MiB / 70.40 MiB [---------------------------------------------->] 100.00% 40.95 MiB p/s ETA 0s70.40 MiB / 70.40 MiB [---------------------------------------------->] 100.00% 39.99 MiB p/s ETA 0s70.40 MiB / 70.40 MiB [---------------------------------------------->] 100.00% 39.99 MiB p/s ETA 0s70.40 MiB / 70.40 MiB [---------------------------------------------->] 100.00% 39.99 MiB p/s ETA 0s70.40 MiB / 70.40 MiB [---------------------------------------------->] 100.00% 37.41 MiB p/s ETA 0s70.40 MiB / 70.40 MiB [---------------------------------------------->] 100.00% 37.41 MiB p/s ETA 0s70.40 MiB / 70.40 MiB [---------------------------------------------->] 100.00% 37.41 MiB p/s ETA 0s70.40 MiB / 70.40 MiB [---------------------------------------------->] 100.00% 35.00 MiB p/s ETA 0s70.40 MiB / 70.40 MiB [---------------------------------------------->] 100.00% 35.00 MiB p/s ETA 0s70.40 MiB / 70.40 MiB [-------------------------------------------------] 100.00% 21.72 MiB p/s 3.4s2025-09-13T21:07:40Z   INFO     [vulndb] Artifact successfully downloaded       repo="mirror.gcr.io/aquasec/trivy-db:2"
2025-09-13T21:07:40Z    INFO    [vuln] Vulnerability scanning is enabled
2025-09-13T21:07:40Z    INFO    [secret] Secret scanning is enabled
2025-09-13T21:07:40Z    INFO    [secret] If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2025-09-13T21:07:40Z    INFO    [secret] Please see https://trivy.dev/v0.66/docs/scanner/secret#recommendation for faster secret detection
2025-09-13T21:07:42Z    INFO    [python] Licenses acquired from one or more METADATA files may be subject to additional terms. Use `--debug` flag to see all affected packages.
2025-09-13T21:07:43Z    INFO    Detected OS     family="debian" version="11.11"
2025-09-13T21:07:43Z    INFO    [debian] Detecting vulnerabilities...   os_version="11" pkg_num=114
2025-09-13T21:07:43Z    INFO    Number of language-specific files       num=1
2025-09-13T21:07:43Z    INFO    [python-pkg] Detecting vulnerabilities...
2025-09-13T21:07:43Z    WARN    Using severities from other vendors for some vulnerabilities. Read https://trivy.dev/v0.66/docs/scanner/vulnerability#severity-selection for details.
2025-09-13T21:07:43Z    INFO    Table result includes only package filenames. Use '--format json' option to get the full path to the package file.

Report Summary

┌──────────────────────────────────────────────────────────────────────────────────┬────────────┬─────────────────┬─────────┐
│                                      Target                                      │    Type    │ Vulnerabilities │ Secrets │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ secure-flask-app (debian 11.11)                                                  │   debian   │       150       │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.9/site-packages/MarkupSafe-3.0.2.dist-info/METADATA        │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.9/site-packages/blinker-1.9.0.dist-info/METADATA           │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.9/site-packages/click-8.1.8.dist-info/METADATA             │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.9/site-packages/flask-2.3.3.dist-info/METADATA             │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.9/site-packages/importlib_metadata-8.7.0.dist-info/METADA- │ python-pkg │        0        │    -    │
│ TA                                                                               │            │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.9/site-packages/itsdangerous-2.2.0.dist-info/METADATA      │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.9/site-packages/jinja2-3.1.6.dist-info/METADATA            │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.9/site-packages/pip-25.2.dist-info/METADATA                │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.9/site-packages/setuptools-58.1.0.dist-info/METADATA       │ python-pkg │        3        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.9/site-packages/werkzeug-2.3.7.dist-info/METADATA          │ python-pkg │        4        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.9/site-packages/wheel-0.45.1.dist-info/METADATA            │ python-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ usr/local/lib/python3.9/site-packages/zipp-3.23.0.dist-info/METADATA             │ python-pkg │        0        │    -    │
└──────────────────────────────────────────────────────────────────────────────────┴────────────┴─────────────────┴─────────┘
Legend:
- '-': Not scanned
- '0': Clean (no security findings detected)


secure-flask-app (debian 11.11)
===============================
Total: 150 (UNKNOWN: 1, LOW: 99, MEDIUM: 31, HIGH: 14, CRITICAL: 5)


---------------------------------------------------------------------------------------------------------------
- step 2: create security check script
  
  @echo off
echo === DOCKER SECURITY HARDENING VERIFICATION ===

echo [1] Verifying non-root user execution...
docker exec flask-app whoami

echo [2] Checking container security options...
docker inspect flask-app --format "{{.HostConfig.SecurityOpt}}"

echo [3] Testing application endpoints...
curl http://localhost:5000/health
curl http://localhost:5000/info

echo [4] Checking running processes...
docker exec flask-app ps aux

echo === SECURITY VERIFICATION COMPLETE ===
pause
-------------------------------------------------------------------------------------------------------------------------
- step 3: do security checks by checking the health of the application
  curl http://localhost:5000/health

  --------------------------------------------------------------------------------------------------------------------------------
  
OVERALL STRUCTIRE:
secure-docker-app/
├── Dockerfile              # Secure container configuration
├── .dockerignore          # Files to exclude from build
├── app.py                 # Flask application
├── requirements.txt       # Python dependencies
├── docker-compose.yml     # Development environment
├── security-check.bat     # Windows security verification
└── README.md             # This documentation
  
