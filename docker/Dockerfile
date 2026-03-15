# Use official Python image
FROM python:3.12-slim

# Set working directory
WORKDIR /app

# Copy application code
COPY app/ .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Expose port
EXPOSE 5000

# Run the application
CMD ["python", "app.py"]