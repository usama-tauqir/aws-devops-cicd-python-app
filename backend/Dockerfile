# =========================
# Stage 1: Build dependencies
# =========================

FROM python:3.9-slim AS builder

WORKDIR /app

COPY requirements.txt .

RUN pip install --upgrade pip && \
    pip install --no-cache-dir --prefix=/install -r requirements.txt


# =========================
# Stage 2: Production image
# =========================

FROM python:3.9-slim

WORKDIR /app

COPY --from=builder /install /usr/local

COPY . .

EXPOSE 8000

CMD ["python","manage.py","runserver","0.0.0.0:8000"]








































# # Use Python 3.9 because this project uses Django 3.2.4
# FROM python:3.9-slim

# # Prevent Python from writing .pyc files
# ENV PYTHONDONTWRITEBYTECODE=1

# # Prevent Python buffering logs
# ENV PYTHONUNBUFFERED=1

# # Set working directory
# WORKDIR /app

# # Copy requirements file
# COPY requirements.txt .

# # Install dependencies
# RUN pip install --upgrade pip && \
#     pip install --no-cache-dir -r requirements.txt

# # Copy backend source code
# COPY . .

# # Expose Django port
# EXPOSE 8000

# # Run Django server
# CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]