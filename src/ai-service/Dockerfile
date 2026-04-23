# Use the official Python image as the base image (Alpine for minimal attack surface)
FROM python:3.12.7-alpine

# Add metadata labels for security and traceability
LABEL maintainer="Your Team <team@example.com>" \
      version="0.1.0" \
      description="AI Service Microservice" \
      org.opencontainers.image.source="https://github.com/your-org/aks-store-demo"

# Update packages to apply security patches
RUN apk update && apk upgrade && rm -rf /var/cache/apk/*

# Create a non-root user for running the application
RUN addgroup -g 1001 appuser && adduser -u 1001 -S appuser -G appuser

# Set the working directory to /app
WORKDIR /app

# Set the build argument for the app version number
ARG APP_VERSION=0.1.0

# Copy only the requirements file first (for better layer caching)
COPY requirements.txt .

# Install dependencies as root, then clean up
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt && \
    rm -rf /root/.cache

# Copy only necessary application files (avoid copying sensitive or unnecessary files)
COPY main.py .
COPY routers/ ./routers/

# Change ownership of the app directory to the non-root user
RUN chown -R appuser:appuser /app

# Switch to the non-root user
USER appuser

# Expose port 5001 for the FastAPI application
EXPOSE 5001

# Set the environment variable for the app version number
ENV APP_VERSION=$APP_VERSION

# Add a health check (adjust URL/path based on your app's endpoints)
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:5001/health || exit 1

# Start the FastAPI application
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "5001"]