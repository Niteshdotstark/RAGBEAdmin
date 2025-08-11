# --- Stage 1: Build dependencies ---
FROM python:3.11-slim as builder

# Set working directory
WORKDIR /app

# Install system dependencies
# This is a large set of dependencies for Chrome and other libraries, 
# which is why we put it in the builder stage.
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    build-essential \
    libpq-dev \
    python3-dev \
    wget \
    unzip \
    gnupg \
    libglib2.0-0 \
    libnss3 \
    libgconf-2-4 \
    libfontconfig1 \
    libxrender1 \
    libxi6 \
    libxrandr2 \
    libxss1 \
    libxtst6 \
    libgtk-3-0 \
    libgbm-dev \
    libasound2 \
    xvfb \
    && rm -rf /var/lib/apt/lists/*

# Install Google Chrome in the builder stage
RUN wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - && \
    echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends google-chrome-stable && \
    rm -rf /var/lib/apt/lists/*

# Copy requirements file first to leverage Docker cache
COPY requirements.txt .

# Install Python dependencies
# This step is where the "no space" error occurred, now it's isolated
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

# --- Stage 2: Create a production-ready image ---
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Install only essential runtime dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Copy installed packages from the builder stage
COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages

# Copy the application code from the current directory
COPY . .

# Create and set permissions for uploads directory
RUN mkdir -p /app/uploads && chmod -R 777 /app/uploads

# Expose port
EXPOSE 8000

# Run with uvicorn
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]