# base defines a base stage that uses the official python runtime base image
FROM python:3.11-slim AS base

# Add curl for healthcheck
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

# Set the application directory
WORKDIR /usr/local/app

# Install our requirements.txt
COPY requirements.txt ./requirements.txt
RUN pip install --no-cache-dir -r requirements.txt && \
    pip uninstall -y setuptools wheel

# create non-root user (in base so dev and final both get it)
RUN addgroup --system appuser && adduser --system --ingroup appuser appuser

# dev defines a stage for development, where it'll watch for filesystem changes
FROM base AS dev
RUN pip install watchdog
ENV FLASK_ENV=development
USER appuser
CMD ["python", "app.py"]

# final defines the stage that will bundle the application for production
FROM base AS final
COPY --chown=appuser:appuser . .
EXPOSE 5000
USER appuser

# Copy our code from the current folder to the working directory inside the container
COPY . .


# Define our command to be run when launching the container
CMD ["gunicorn", "app:app", "-b", "0.0.0.0:5000", "--log-file", "-", "--access-logfile", "-", "--workers", "4", "--keep-alive", "0"]
