# AWS DevOps CI/CD Project — Django + React E-Commerce

A full-stack e-commerce application built with **Django** and **React**, packaged and deployed using a DevOps workflow centered on **AWS, Docker, CI/CD, Linux, Nginx, Git, and GitHub**.

This repository is intended to showcase both the application itself and the DevOps practices used to build, deploy, and operate it.

---

## Project Overview

The application allows users to:

- Browse products and view product details
- Register and log in
- Manage account information
- Add, update, and delete addresses
- Manage payment cards
- Complete checkout using Stripe
- View order history
- Delete their account
- Use JWT-based authentication

Admin users can also:

- Add products
- Edit product details
- Delete products
- View customer orders
- Update delivery/order status
- Search orders by customer, address, or product

---

## DevOps Focus

This project is structured to demonstrate practical DevOps skills including:

- **AWS** cloud deployment
- **Docker** containerization
- **CI/CD** automation with GitHub Actions / Jenkins
- **Linux** server administration
- **Nginx** web server / reverse proxy configuration
- **Git & GitHub** version control and collaboration
- Application deployment and troubleshooting
- Secure handling of environment variables and secrets

---

## Tech Stack

### DevOps / Infrastructure

- AWS
- Docker
- GitHub Actions / Jenkins
- Linux
- Nginx
- Git
- GitHub

### Backend

- Python
- Django

### Frontend

- React
- JavaScript
- npm

### Authentication & Payments

- JSON Web Tokens (JWT)
- Stripe

---

## High-Level Architecture

```text
Developer
   |
   v
GitHub Repository
   |
   v
CI/CD Pipeline
(GitHub Actions / Jenkins)
   |
   v
Docker Build
   |
   v
AWS Deployment Environment
   |
   v
Linux Server
   |
   v
Nginx
   |
   v
Django Backend + React Frontend
```

> Update this diagram if your actual AWS architecture uses specific services such as EC2, ECS, ECR, RDS, S3, CloudFront, or a load balancer.

---

## CI/CD Workflow

A typical workflow for this project is:

1. Developer pushes code to GitHub.
2. CI/CD workflow starts automatically.
3. Application dependencies are installed.
4. Build and validation steps are executed.
5. Docker image is built.
6. Application is deployed to the AWS environment.
7. Nginx serves or proxies application traffic.
8. Deployment is verified.

> Keep only the steps that match your actual implementation.

---

## Docker

Docker is used to make the application easier to build and run consistently across environments.

Recommended repository structure:

```text
.
├── backend/
├── frontend/
├── Dockerfile
├── docker-compose.yml
├── .github/
│   └── workflows/
├── nginx/
├── docs/
│   └── screenshots/
└── README.md
```

If your repository uses separate Dockerfiles for frontend and backend, document them here.

---

## AWS Deployment

This project is deployed to AWS as part of the DevOps workflow.

Document your real AWS resources here, for example:

- Compute service used
- Networking configuration
- Security groups
- Container registry, if used
- Database service, if used
- Domain / DNS configuration
- HTTPS / TLS configuration
- Monitoring and logging

> Do not list AWS services here unless they are actually used in this project.

---

## Nginx

Nginx can be used as a web server and/or reverse proxy in front of the application.

Typical responsibilities include:

- Serving frontend assets
- Proxying API requests to Django
- Handling application routing
- Supporting HTTPS
- Managing production traffic

Add the relevant Nginx configuration file to the repository if it is part of your deployment.

---

## Security & Secrets

Never commit secrets directly to GitHub.

Keep sensitive values in environment variables or secure secret stores.

Examples:

```env
DJANGO_SECRET_KEY=
STRIPE_SECRET_KEY=
STRIPE_PUBLISHABLE_KEY=
DATABASE_URL=
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
```

Make sure files such as these are ignored:

```gitignore
.env
*.pem
*.key
__pycache__/
node_modules/
```

If any secret has ever been committed to Git history, rotate it immediately.

---

## Application Features

### Product Management

Users can browse available products and open individual product detail pages.

Admins can:

- Add products
- Update product information
- Change stock status
- Delete products

### Authentication

The application uses JWT-based authentication.

Authentication checks are performed for protected requests, while public product-list and product-detail endpoints can remain accessible without authentication.

### Checkout & Stripe

Users can:

- Select products
- Manage delivery addresses
- Add or update Stripe card information
- Complete payment
- View payment confirmation

### Orders

Users can view their own orders.

Admins can:

- View all orders
- Update delivery status
- Search orders

### Account Management

Users can:

- Update profile information
- Reset their password
- Delete their account

---

## Local Development Setup

### 1. Clone the repository

```bash
git clone <YOUR_REPOSITORY_URL>
cd <YOUR_REPOSITORY_FOLDER>
```

---

### 2. Backend Setup

Move into the backend directory:

```bash
cd backend
```

Create a virtual environment:

#### Linux / macOS

```bash
python3 -m venv env
source env/bin/activate
```

#### Windows

```bash
python -m venv env
env\Scripts\activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Add the required environment variables.

Run the backend:

```bash
python manage.py runserver
```

---

### 3. Frontend Setup

Open another terminal and move into the frontend directory:

```bash
cd frontend
```

Install dependencies:

```bash
npm install
```

Start the frontend:

```bash
npm start
```

---

## Screenshots

Replace these placeholders with screenshots from **your own repository/project**.

Recommended screenshots:

1. Application home / products page
2. GitHub Actions or Jenkins pipeline
3. Successful Docker build
4. Running containers
5. AWS deployment
6. Nginx configuration / reverse proxy
7. Final deployed application

Example:

```markdown
![CI/CD Pipeline](docs/screenshots/cicd-pipeline.png)
![AWS Deployment](docs/screenshots/aws-deployment.png)
![Application](docs/screenshots/application.png)
```

---

## Challenges & Solutions

Use this section to document real engineering problems you faced.

Example format:

### Challenge 1 — Deployment Configuration

**Problem:**  
Describe the deployment issue.

**Solution:**  
Explain how you investigated and fixed it.

### Challenge 2 — CI/CD Failure

**Problem:**  
Describe the pipeline failure.

**Solution:**  
Explain how you debugged and resolved it.

### Challenge 3 — Nginx / Application Routing

**Problem:**  
Describe the routing or proxy issue.

**Solution:**  
Explain the final configuration.

---

## What I Learned

This project helped me strengthen practical skills in:

- Cloud deployment on AWS
- Docker containerization
- CI/CD automation
- Linux server administration
- Nginx configuration
- Git and GitHub workflows
- Django and React deployment
- Troubleshooting deployment issues
- Environment-variable and secret management
- Building a complete application deployment workflow

---

## Future Improvements

Possible next steps:

- Infrastructure as Code with Terraform
- Kubernetes deployment
- Prometheus and Grafana monitoring
- Automated rollback strategy
- Centralized logging
- HTTPS automation
- Automated testing in CI/CD
- Production-grade secrets management

Only mark these as completed once they are actually implemented.

---

## Repository Checklist

Before featuring this project on LinkedIn, make sure:

- [ ] Repository is public
- [ ] No secrets, passwords, `.env`, or `.pem` files are committed
- [ ] README describes the actual implementation
- [ ] Screenshots belong to this repository/project
- [ ] CI/CD workflow is included
- [ ] Docker configuration is included
- [ ] AWS deployment is documented
- [ ] Nginx configuration is documented if used
- [ ] Repository topics are added
- [ ] Project has a clear description
- [ ] LinkedIn Featured section links to this repository

---

## Author

**Muhammad Usama Tauqir**

DevOps Engineer Intern  
AWS | Docker | Kubernetes | Terraform | CI/CD | Linux

- LinkedIn: https://www.linkedin.com/in/muhammad-usama-tauqir/
- GitHub: `<ADD_YOUR_GITHUB_PROFILE_URL>`

---

## Note

This project is built for learning, hands-on practice, and demonstrating practical full-stack and DevOps skills.

If you use this repository as a portfolio project, keep the documentation aligned with the tools and infrastructure that are actually implemented.
