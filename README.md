# AI Digital Twin: Lucas Maciel Sousa 🤖

[![Next.js](https://img.shields.io/badge/Frontend-Next.js%2016-black?style=flat-square&logo=next.dotjs)](https://nextjs.org/)
[![FastAPI](https://img.shields.io/badge/Backend-FastAPI-009688?style=flat-square&logo=fastapi)](https://fastapi.tiangolo.com/)
[![AWS](https://img.shields.io/badge/Cloud-AWS-FF9900?style=flat-square&logo=amazon-aws)](https://aws.amazon.com/)
[![Terraform](https://img.shields.io/badge/Infrastructure-Terraform-7B42BC?style=flat-square&logo=terraform)](https://www.terraform.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)

An interactive, high-performance, serverless digital twin of **Lucas Maciel Sousa**, designed to answer questions about his career, education, specialties, and professional background. The twin communicates professionally and approachably, powered by **AWS Bedrock** and curated information dynamically extracted from his professional records (LinkedIn profile, personal summaries, and fact files).

---

## 🏗️ Architecture Overview

The application is deployed on AWS using a serverless, highly-scalable architecture designed for maximum performance, minimal cost, and high availability.

```text
                      ┌──────────────────┐
                      │   User Browser   │
                      └────────┬─────────┘
                               │
             ┌─────────────────┴─────────────────┐
     (Static Assets)                    (API Requests /chat)
             │                                   │
    ┌────────▼────────┐                 ┌────────▼────────┐
    │  AWS CloudFront │                 │   AWS API       │
    │  Distribution   │                 │   Gateway       │
    └────────┬────────┘                 └────────┬────────┘
             │                                   │
    ┌────────▼────────┐                 ┌────────▼────────┐
    │   S3 Frontend   │                 │   AWS Lambda    │
    │     Bucket      │                 │ (FastAPI/Mangum)│
    └─────────────────┘                 └────┬───────┬────┘
                                             │       │
                               ┌─────────────┴──┐ ┌──┴────────────┐
                               │  AWS Bedrock   │ │   S3 Memory   │
                               │  (Nova Model)  │ │    Bucket     │
                               └────────────────┘ └───────────────┘
```

### Key Components

- **Frontend**: A highly responsive, single-page application built with **Next.js 16 (App Router)** and **Tailwind CSS v4**. It serves as an elegant chat interface featuring fluid animations, loading states, and responsive styling.
- **Backend**: A **FastAPI** web framework running in **AWS Lambda** via **Mangum**. It features CORS middleware, session-based conversation persistence, dynamic prompt assembling, and direct integration with Amazon Bedrock.
- **AI Model**: **Amazon Bedrock**'s latest models (e.g., `amazon.nova-lite-v1:0` or configurable alternatives), accessed through the `converse` API for low-latency, conversational intelligence.
- **Memory Storage**: Dynamic session persistence using **AWS S3** in production (scoped to unique session IDs) and standard file storage locally, preserving up to 50 historical messages.
- **Infrastructure as Code (IaC)**: A fully declarative **Terraform** configuration mapping the entire AWS ecosystem.
- **CI/CD**: Fully automated deployment pipelines via **GitHub Actions** backing multi-environment workspaces (`dev`, `test`, `prod`).

---

## 📂 Project Structure

```text
twin/
├── .github/workflows/    # CI/CD pipelines
│   ├── deploy.yml        # Automatically deploys frontend and backend via Terraform
│   └── destroy.yml       # Manual workflow to tear down the environment
├── backend/              # FastAPI application
│   ├── data/             # Profile data sources for Lucas
│   │   ├── facts.json    # Structured metadata (education, skills, etc.)
│   │   ├── linkedin.pdf  # LinkedIn resume PDF (dynamically parsed)
│   │   ├── style.txt     # Communication style directives
│   │   └── summary.txt   # Short professional profile summary
│   ├── context.py        # Prompts compiler and system directives
│   ├── deploy.py         # Docker-based Lambda dependencies packaging utility
│   ├── lambda_handler.py # Mangum Lambda handler
│   ├── pyproject.toml    # Modern python project definition (uv format)
│   ├── requirements.txt  # Core Lambda dependencies
│   ├── resources.py      # PDF parsing and file loading operations
│   └── server.py         # Main FastAPI routes, Bedrock interface, and memory manager
├── frontend/             # Next.js Application
│   ├── app/              # Next.js page layouts and fonts
│   ├── components/       # React interactive chat panel component (twin.tsx)
│   ├── public/           # Static assets (avatar.png, icons)
│   ├── package.json      # Node scripts and UI dependencies
│   └── postcss.config.mjs # PostCSS config (Tailwind CSS v4 integration)
├── scripts/              # Bash helper utilities
│   ├── deploy.sh         # Executes deployment (Docker -> Terraform -> Next.js Build -> S3 sync)
│   └── destroy.sh        # Performs complete cloud resources cleanup
└── terraform/            # Infrastructure-as-Code declarations
    ├── main.tf           # Declares S3 buckets, IAM roles, API Gateway, CloudFront, Lambda
    ├── variables.tf      # Configuration variables (timeout, model ID, domain settings)
    └── terraform.tfvars  # Global default variables values
```

---

## 🚀 Local Development Setup

Follow these steps to run the application locally on your machine.

### Prerequisites

Ensure you have the following installed:
- **Python 3.12+** with [Astral uv](https://github.com/astral-sh/uv)
- **Node.js v20+** & **npm**
- **AWS CLI** configured with credentials that have Bedrock access
- **Docker** — only needed when packaging the Lambda zip (`backend/deploy.py`), not for local dev
- **Terraform** — only needed for provisioning AWS infrastructure

### 1. Backend Setup

1. Navigate to the backend directory:
   ```bash
   cd backend
   ```

2. Create a virtual environment and install dependencies:
   ```bash
   uv sync
   ```

3. Configure your local environment variables in a `.env` file:
   ```ini
   DEFAULT_AWS_REGION=us-east-1
   BEDROCK_MODEL_ID=amazon.nova-lite-v1:0
   USE_S3=false
   MEMORY_DIR=../memory
   CORS_ORIGINS=http://localhost:3000
   ```

4. Start the FastAPI development server:
   ```bash
   uv run server.py
   ```
   The backend API will now be running at `http://localhost:8000`. You can inspect the interactive docs at `http://localhost:8000/docs`.

### 2. Frontend Setup

1. Open a new terminal and navigate to the frontend directory:
   ```bash
   cd frontend
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Run the Next.js development server:
   ```bash
   npm run dev
   ```
   Open `http://localhost:3000` in your browser. The frontend is configured to automatically communicate with `http://localhost:8000`.

---

## ☁️ Infrastructure & Cloud Deployment

All infrastructure is provisioned inside AWS using **Terraform**. 

### Configuration Variables

Customize your deployments by editing `terraform/terraform.tfvars` or creating a custom `terraform/prod.tfvars` file:

| Variable | Description | Default |
| :--- | :--- | :--- |
| `project_name` | Identifier prefixed to all resources. | `twin` |
| `environment` | Deployment workspace (`dev`, `test`, `prod`). | `dev` |
| `bedrock_model_id` | AWS Bedrock model identifier. | `amazon.nova-lite-v1:0` |
| `lambda_timeout` | Lambda function timeout in seconds. | `60` |
| `api_throttle_burst_limit` | API Gateway throttle burst limit. | `10` |
| `api_throttle_rate_limit` | API Gateway throttle rate limit (req/s). | `5` |
| `use_custom_domain` | Set to `true` to provision DNS and SSL records. | `false` |
| `root_domain` | Apex domain name (e.g. `lucasmaciel.com`). | `""` |

### Manual Deployment Script

An all-in-one deployment script compiles the backend, provisions infrastructure, compiles the frontend bundle, and deploys everything:

```bash
# Set up AWS credentials and default variables
export DEFAULT_AWS_REGION=us-east-1

# Deploy to 'dev' environment
chmod +x scripts/deploy.sh
./scripts/deploy.sh dev
```

The script will automate:
1. Compiling Python dependencies inside a Lambda-matching Docker container (`public.ecr.aws/lambda/python:3.12`) to avoid OS-level compilation conflicts.
2. Initializing and selecting the corresponding Terraform workspace.
3. Applying the infrastructure configuration on AWS.
4. Extracting the generated API Gateway endpoints.
5. Building the Next.js frontend with the static API endpoint embedded.
6. Syncing the compiled Next.js output directory (`frontend/out`) directly to the public S3 bucket behind CloudFront.

### Manual Destruction Script

To completely clean up and destroy all resources created in a specific workspace to avoid costs:

```bash
chmod +x scripts/destroy.sh
./scripts/destroy.sh dev
```

---

## 🤖 CI/CD Integration (GitHub Actions)

The repository comes equipped with automated pipelines located in `.github/workflows/`:

- **Deployment (`deploy.yml`)**: Triggered automatically on `push` events to the `main` branch, or via manual trigger (`workflow_dispatch`). It authenticates with AWS using OpenID Connect (OIDC), sets up the required environment (Python, Terraform, Node.js, `uv`), executes `./scripts/deploy.sh`, and automatically invalidates the AWS CloudFront cache to deploy updates instantaneously.
- **Destruction (`destroy.yml`)**: A manually triggered pipeline to safely tear down all cloud infrastructure from a selected workspace.

> [!IMPORTANT]
> To configure the CI/CD pipeline, the following GitHub Secrets must be set up in your repository:
> - `AWS_ROLE_ARN`: The ARN of the AWS IAM Role that GitHub Actions will assume via OIDC.
> - `AWS_ACCOUNT_ID`: Your 12-digit AWS Account ID.
> - `DEFAULT_AWS_REGION`: The target deployment region (e.g., `us-east-1`).
