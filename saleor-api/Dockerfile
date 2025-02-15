# Base image
FROM mcr.microsoft.com/devcontainers/python:3.12

# Set the static URL as a build argument
ARG STATIC_URL

# Define the environment variable with a default value
ENV STATIC_URL=${STATIC_URL:-/static/}

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gettext \
    libffi8 \
    libgdk-pixbuf2.0-0 \
    liblcms2-2 \
    libopenjp2-7 \
    libssl3 \
    libtiff6 \
    libwebp7 \
    shared-mime-info \
    mime-support \
    libpq5 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

COPY . /app

    # Set working directory
WORKDIR /app

# Install Poetry using pip
RUN pip install poetry==1.8.4

# Configure Poetry to disable virtual environments
RUN poetry config virtualenvs.create false

# Copy pyproject.toml and poetry.lock into the image
COPY pyproject.toml poetry.lock ./

# Install Python dependencies using Poetry
RUN poetry install --no-root

# Add a non-root user and group
RUN groupadd -r saleor && useradd -r -g saleor saleor

# Set ownership of the /app directory
RUN mkdir -p /app && chown -R saleor:saleor /app

# Switch to the non-root user
USER saleor

# Collect static files
RUN SECRET_KEY=dummy STATIC_URL=${STATIC_URL} python3 manage.py collectstatic --no-input

# Expose port and command to run the app
EXPOSE 8000
CMD ["gunicorn", "--bind", ":8000", "--workers", "4", "--worker-class", "saleor.asgi.gunicorn_worker.UvicornWorker", "saleor.asgi:application"]
