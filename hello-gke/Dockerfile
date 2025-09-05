FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update --fix-missing && \
    # Install HTTPS transport and certificates to ensure package sources work
    apt-get install -y apt-transport-https ca-certificates && \
    # Install Python 3 and pip
    apt-get install -y python3 python3-pip && \
    # Clean up apt cache
    apt-get clean && \
    # Remove apt lists to reduce image size and avoid stale package lists
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY requirements.txt .
RUN pip3 install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 8080
CMD ["python3", "app.py"]