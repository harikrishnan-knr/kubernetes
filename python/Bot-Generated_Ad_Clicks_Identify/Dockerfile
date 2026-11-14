
# Stage 1: Build dependencies
FROM python:3.12-slim AS builder

WORKDIR /app

COPY FRONTEND/requirements.txt .

RUN pip install \
    --prefix=/install \
    --no-cache-dir \
    --default-timeout=300 \
    --retries=10 \
    -r requirements.txt


# Stage 2: Production image
FROM python:3.12-slim

WORKDIR /app

# Copy installed dependencies
COPY --from=builder /install /usr/local

# Copy frontend application
COPY FRONTEND/ .

# Flask/Gunicorn port
EXPOSE 5000

# Start application
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]

