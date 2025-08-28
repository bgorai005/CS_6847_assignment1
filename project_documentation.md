## Project Documentation: Cloud Performance Testing

### 1. Project Overview

This project is designed to test the performance of a simple web service under various loads. It consists of three main parts:

1.  **A Flask Web Service:** A simple Python-based web application that serves as the service to be tested.
2.  **A Test Client:** A Python script that sends a configurable number of requests per second to the web service to simulate load.
3.  **Kubernetes Configuration:** A set of files to deploy and manage the web service in a Kubernetes cluster, allowing for scalability and high availability.

The primary goal is to understand how the web service performs under different conditions and how containerization and orchestration technologies like Docker and Kubernetes can be used to improve its performance and reliability.

### 2. Core Components

*   **`app/` directory:** This contains the source code for the Flask web service.
    *   `app.py`: The main application file.
    *   `requirements.txt`: A list of Python libraries the application needs to run.
    *   `Dockerfile`: Instructions for building a Docker container image of the application.

*   **`client/` directory:** This contains the script for testing the web service.
    *   `client.py`: The main client script that sends requests to the service.
    *   `utils.py`: Helper functions for the client, such as logging results.

*   **`kubernetes/` directory:** This contains the configuration files for deploying the application to Kubernetes.
    *   `deployment.yaml`: Defines how to deploy the application, including the number of replicas (instances) to run.
    *   `service.yaml`: Exposes the application to the network so it can be accessed.
    *   `hpa.yaml`: (Horizontal Pod Autoscaler) Automatically scales the number of application instances up or down based on CPU usage.

### 3. How it Works (The "Intuition")

Imagine you have a small shop (the **Flask web service**). Customers (requests from the **test client**) come to your shop to get something.

1.  **The Shop (`app/app.py`):** Your shop is very simple. It just says "Hello" to every customer and tells them the current time.

2.  **The Customers (`client/client.py`):** You want to see how your shop handles a lot of customers. So, you hire a group of people to go to your shop at a certain rate (e.g., 10 customers per second). These people are your test client. They record how long it takes for the shop to serve them.

3.  **Putting the Shop in a Box (`app/Dockerfile`):** To make your shop easily movable and consistent, you package it into a standardized box (a **Docker container**). This box contains everything your shop needs to run: the code, the libraries, and the instructions on how to start it. Now you can easily create identical copies of your shop anywhere.

4.  **Managing Many Shops (`kubernetes/`):** What if you have too many customers for one shop? You need to open more shops. This is where **Kubernetes** comes in. It's like a manager for your shops.
    *   `deployment.yaml`: You tell the manager, "I want to have 3 shops running at all times." The manager will make sure there are always 3 identical shops (containers) running.
    *   `service.yaml`: You give the manager a single phone number. When a customer calls this number, the manager directs them to one of the available shops. This way, customers don't need to know the individual address of each shop.
    *   `hpa.yaml`: You tell the manager, "If the workers in my shops are getting too busy (CPU usage is high), automatically open more shops. If they are not busy, close some of the extra shops to save money." This is **autoscaling**.

### 4. The Application (`app/app.py`)

The application is a very basic Flask web service.

*   It has a single endpoint: `/`.
*   When a request is made to this endpoint, it returns a JSON response with a "Hello" message and the current timestamp.
*   It's designed to be lightweight and simple, so we can focus on the performance of the infrastructure around it.

### 5. Containerization (`app/Dockerfile`)

The `Dockerfile` is a set of instructions to build a Docker image.

1.  `FROM python:3.11-slim`: It starts with a lightweight, official Python image.
2.  `WORKDIR /app`: It sets the working directory inside the container to `/app`.
3.  `COPY requirements.txt .`: It copies the `requirements.txt` file into the container.
4.  `RUN pip install ...`: It installs the Python libraries listed in `requirements.txt`.
5.  `COPY . .`: It copies the rest of the application code (i.e., `app.py`) into the container.
6.  `EXPOSE 5000`: It informs Docker that the application will listen on port 5000.
7.  `CMD ["python", "app.py"]`: It specifies the command to run when the container starts.

### 6. The Client (`client/client.py`)

The client script is used to send requests to the web service and measure its performance.

*   It can send requests at a specified rate (e.g., `--rate 50` for 50 requests per second).
*   It has two modes:
    *   **Synchronous (`run_sync`):** For lower request rates (<= 100), it sends one request, waits for the response, and then sends the next. This is like customers forming a single line.
    *   **Asynchronous (`run_async`):** For higher request rates (> 100), it sends many requests at once without waiting for each one to complete. This is like many customers rushing the shop at the same time.
*   It logs the response time for each request to an output file and calculates the average response time.

### 7. Deployment (`kubernetes/`)

The Kubernetes files define how to run the application in a cluster.

*   **`deployment.yaml`:**
    *   `replicas: 3`: It tells Kubernetes to run 3 instances (pods) of our application container.
    *   `image: flask-app:latest`: It specifies the Docker image to use.
    *   `resources`: It sets limits on how much CPU and memory each container can use. This is important for resource management in a shared cluster.

*   **`service.yaml`:**
    *   `type: NodePort`: It makes the service accessible from outside the Kubernetes cluster.
    *   `selector: app: flask-service`: It tells the service which pods to send traffic to (the ones with the label `app: flask-service`).
    *   `port: 5000`: The port the service listens on.
    *   `targetPort: 5000`: The port on the container to send traffic to.

*   **`hpa.yaml`:**
    *   `scaleTargetRef`: It specifies that the HPA should manage the `flask-deployment`.
    *   `minReplicas: 3`, `maxReplicas: 10`: It sets the minimum and maximum number of pods.
    *   `averageUtilization: 50`: It tells the HPA to scale up the number of pods if the average CPU usage across all pods goes above 50%, and scale down if it's below 50%.

### 8. Pros and Cons

#### Pros:

*   **Scalability:** With Kubernetes, it's easy to scale the application horizontally (by adding more instances) to handle more traffic. The HPA makes this process automatic.
*   **High Availability:** If one of the application instances crashes, Kubernetes will automatically restart it or start a new one, ensuring the service remains available.
*   **Portability:** Docker containers can run on any machine that has Docker installed, making it easy to move the application from a developer's laptop to a production server.
*   **Consistency:** Docker ensures that the application runs in the same environment every time, which helps to eliminate "it works on my machine" problems.
*   **Resource Efficiency:** Containers are more lightweight than virtual machines, so you can run more of them on a single server.

#### Cons:

*   **Complexity:** For a simple application, this setup can be overkill. Docker and Kubernetes have a learning curve.
*   **Overhead:** While containers are lightweight, there is still some overhead associated with running Docker and Kubernetes.
*   **State Management:** This architecture is best suited for stateless applications (like our "Hello" service). If the application needed to store data (like user sessions or a database), managing that state across multiple instances would add significant complexity.

### 9. Step-by-Step Instructions

The `README.md` file in the project provides detailed instructions on how to run the project. Here is a summary:

1.  **Run the app locally:** You can run the Flask app directly on your machine to test it.
2.  **Run the app with Docker:** You can build a Docker image and run the app in a container.
3.  **Run the test client:** You can use the client to send requests to the app (either running locally or in Docker) and see the performance results.
4.  **Deploy to Kubernetes:** You can deploy the app to a Kubernetes cluster (like Minikube) and use the client to test its performance in a more realistic, scalable environment.
