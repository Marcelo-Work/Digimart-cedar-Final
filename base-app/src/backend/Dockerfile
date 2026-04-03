FROM python:3.13-slim

FROM python:3.11-slim as builder

WORKDIR /app

RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    python3-dev \
    libpq-dev \
    curl \
    && rm -rf /var/lib/apt/lists/*


RUN pip install --upgrade pip

COPY src/backend/requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt


FROM python:3.11-slim
RUN useradd -m -u 1000 appuser

WORKDIR /app

# Copy installed dependencies from builder
COPY --from=builder /root/.local /home/appuser/.local
COPY --from=builder /usr/local/bin /usr/local/bin

# Copy application code
COPY src/backend/ .

# Set ownership and permissions
RUN chown -R appuser:appuser /app
USER appuser

# Environment variables
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PATH=/home/appuser/.local/bin:$PATH \
    DJANGO_SETTINGS_MODULE=config.settings

# Install curl for health checks
USER root
RUN apt-get update && apt-get install -y --no-install-recommends curl \
    && rm -rf /var/lib/apt/lists/*
USER appuser

# Create directory for SQLite database
RUN mkdir -p /app/db && chown appuser:appuser /app/db

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD curl -f http://localhost:3000/health/ || exit 1

# Default command (override in docker-compose for dev)
CMD ["python", "manage.py", "runserver", "0.0.0.0:3000"]
RUN chmod +x entrypoint.sh

EXPOSE 3000

ENTRYPOINT ["sh", "entrypoint.sh"]
