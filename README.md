# Cloud Performance Testing: Flask Web Service

This project contains a simple Flask web service, a client to test its performance, and Kubernetes configuration for deployment. The goal is to analyze the performance of the web service under different request loads.

## Project Structure

```
.
├── app/
│   ├── app.py           # Flask web service
│   ├── Dockerfile       # Docker configuration for the app
│   └── requirements.txt # Python dependencies
├── client/
│   ├── client.py        # Test client (sync and async)
│   └── utils.py         # Utility functions for the client
├── kubernetes/
│   ├── deployment.yaml  # Kubernetes deployment for the app
│   ├── hpa.yaml         # Horizontal Pod Autoscaler configuration
│   └── service.yaml     # Kubernetes service for the app
└── results/             # (Generated) Output from the client
```

## Getting Started

### Prerequisites

*   Python 3.8+
*   Docker
*   Kubernetes (e.g., Minikube, Docker Desktop)

### 1. Set up the Flask Application

The `app` directory contains a simple Flask application.

**To run locally:**

1.  Navigate to the `app` directory:
    ```bash
    cd app
    ```
2.  Install the dependencies:
    ```bash
    pip install -r requirements.txt
    ```
3.  Run the Flask app:
    ```bash
    python app.py
    ```
    The server will be running at `http://0.0.0.0:5001`.

**To run with Docker:**

1.  Navigate to the `app` directory:
    ```bash
    cd app
    ```
2.  Build the Docker image:
    ```bash
    docker build -t flask-app:latest .
    ```
3.  Run the Docker container:
    ```bash
    docker run -p 5001:5001 flask-app:latest
    ```
    The server will be accessible at `http://localhost:5001`.

### 2. Run the Test Client

The `client` directory contains a Python script to send requests to the Flask application. It can send both synchronous and asynchronous requests.

1.  Navigate to the `client` directory:
    ```bash
    cd client
    ```
2.  Run the client script with the desired parameters:
    ```bash
    python client.py --target http://localhost:5001 --rate <requests_per_second> --output ../results/output.txt --duration <seconds>
    ```
    *   `--target`: The URL of the Flask application.
    *   `--rate`: The number of requests per second. If the rate is > 100, it will use asynchronous requests.
    *   `--output`: The file to save the results to.
    *   `--duration`: The duration of the test in seconds.

    The results will be saved in the `results` directory.

### 3. Deploy to Kubernetes

The `kubernetes` directory contains the configuration files to deploy the Flask application to a Kubernetes cluster.

1.  Make sure your Docker image is available to your Kubernetes cluster. If you are using Minikube, you can build the image within Minikube's Docker daemon:
    ```bash
    eval $(minikube docker-env)
    cd app
    docker build -t flask-app:latest .
    cd ..
    ```

2.  Apply the Kubernetes configurations:
    ```bash
    kubectl apply -f kubernetes/deployment.yaml
    kubectl apply -f kubernetes/service.yaml
    kubectl apply -f kubernetes/hpa.yaml
    ```

3.  Find the service URL:
    ```bash
    minikube service flask-service --url
    ```

4.  Run the client with the service URL:
    ```bash
    python client/client.py --target <service_url> --rate 10 --output results/kube_output.txt
    ```