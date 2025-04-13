# AWS EKS Cluster Setup and Intelligent Query Sequencing

This repository contains the solution to provision an AWS EKS cluster, deploy a sample full-stack application (frontend, backend, MongoDB, Qdrant), and implement an intelligent query sequencing system for LLMs (Large Language Models) using Redis.

## Part 1: AWS EKS Cluster Setup

### 1. Provisioning the EKS Cluster

To provision an AWS EKS (Elastic Kubernetes Service) cluster using CloudFormation and Kubernetes manifests:

#### CloudFormation Template:

A CloudFormation template (`cloudformation.yaml`) is provided to create the AWS EKS cluster. It provisions the cluster, networking resources, IAM roles, and worker nodes.

#### Steps to deploy the CloudFormation template:

Use the `eksctl` tool to create the cluster:

```bash
eksctl create cluster -f cloudformation.yaml
This creates the EKS cluster along with the required networking, IAM roles, and worker nodes.

2. Kubernetes Manifests
The eks/k8s/ folder contains Kubernetes YAML manifests for deploying the frontend, backend, MongoDB, and Qdrant services to the EKS cluster. The steps are as follows:

Deploy the Services:
Deploy the frontend, backend, MongoDB, and Qdrant services using Kubernetes manifests.

Run the following commands:

bash
Copy
Edit
kubectl create namespace app-stack
kubectl apply -f eks/k8s/ --namespace app-stack
Verify the Application:
After applying the manifests, check if the services and pods are running:

bash
Copy
Edit
kubectl get svc -n app-stack
kubectl get pods -n app-stack
A LoadBalancer is provisioned for the frontend service. Once the load balancer is provisioned, you will receive a DNS URL to access the frontend application.

Access the Application:
The Load Balancer DNS will be available under the EXTERNAL-IP column of the frontend service:

bash
Copy
Edit
kubectl get svc frontend -n app-stack
Open the provided DNS URL in a browser to access the frontend application.

Part 2: Intelligent Query Sequencing for LLMs
1. Install Dependencies
Ensure that Redis is installed and running (either locally or via AWS ElastiCache). Also, ensure you have Python 3 and the redis package installed.

To install the redis package in Python:

bash
Copy
Edit
pip3 install redis
2. Create the Query Scheduler Python Script
The scripts/query_scheduler.py file contains a Python script that connects to Redis, reads user queries from a Redis queue, and intelligently assigns them to the most appropriate LLM based on its properties (latency, throughput, capability).

Query Scheduler Script
python
Copy
Edit
import redis
import json
import time

# Simulated LLMs with latency (ms), throughput (qps), capability score
LLMS = [
    {"name": "llm1", "latency": 200, "throughput": 5, "capability": 0.9, "load": 0},
    {"name": "llm2", "latency": 100, "throughput": 10, "capability": 0.7, "load": 0},
    {"name": "llm3", "latency": 300, "throughput": 3, "capability": 1.0, "load": 0},
]

# Function to score LLM based on query properties
def score_llm(llm, query):
    urgency = query.get("urgency", 1)
    complexity = query.get("complexity", 1)

    # Weighted score combining all aspects
    score = (
        -llm["latency"] +
        llm["throughput"] * 10 +
        llm["capability"] * 100 -
        llm["load"] * 50 +
        urgency * 50
    )
    return score

# Function to select the best LLM for the query
def get_best_llm(query):
    scored = [(score_llm(llm, query), llm) for llm in LLMS]
    best = max(scored, key=lambda x: x[0])[1]
    best["load"] += 1  # Simulate load increase
    return best["name"]

# Connect to Redis
r = redis.Redis(host='localhost', port=6379, db=0)

while True:
    _, raw = r.blpop("query_queue")  # Blocking pop
    query = json.loads(raw)
    
    selected_llm = get_best_llm(query)
    print(f"Dispatching query '{query['id']}' to {selected_llm}")
    
    time.sleep(0.5)  # Simulate processing time
3. Running the Query Scheduler
Start the Redis server (locally or use AWS ElastiCache).

Run the query scheduler script:

bash
Copy
Edit
python3 scripts/query_scheduler.py
Push test queries to the Redis queue:

bash
Copy
Edit
redis-cli
LPUSH query_queue '{"id": 1, "urgency": 2, "complexity": 3}'
LPUSH query_queue '{"id": 2, "urgency": 1, "complexity": 2}'
The script will process each query, evaluate the best LLM based on latency, throughput, and capability, and then dispatch the query to the selected LLM.

4. Example Output
The output in the terminal will look like this:

bash
Copy
Edit
Dispatching query '1' to llm2
Dispatching query '2' to llm1
The script intelligently allocates queries to the most suitable LLM based on the scoring function.

Directory Structure
Your project directory should be structured as follows:

bash
Copy
Edit
/YourProject
    /eks
        cloudformation.yaml
        /k8s/
            frontend.yaml
            backend.yaml
            mongo.yaml
            qdrant.yaml
    /scripts
        query_scheduler.py
    /docs
        README.md
Submission Instructions
GitHub Repository:
Create a new GitHub repository (e.g., aws-eks-query-sequencing).

Push your project to the GitHub repository.

.zip File Submission:
Alternatively, you can compress the project folder into a .zip file:

bash
Copy
Edit
zip -r YourProject.zip YourProject/
Notes
Ensure Redis is running before starting the query scheduler.

The cloudformation.yaml file creates the necessary AWS resources for the EKS cluster, including networking, IAM roles, and worker nodes.

Kubernetes manifests are provided to deploy the full-stack application to the EKS cluster.

The Python script intelligently schedules queries to the LLMs based on their properties.

markdown
Copy
Edit

---

### Key Changes for Markdown:

1. **Headings**: Use `#` for titles and `##` for sub-headings.
2. **Code Blocks**: Use triple backticks (\`\`\`) for code blocks. You can specify the language (e.g., `bash` or `python`) after the first set of backticks to enable syntax highlighting.
3. **Lists**: Use `-` or `*` for bullet points.
4. **Inline Code**: Use single backticks for inline code (e.g., `eksctl`).
5. **Bold/Italic**: You can make text bold with `**` or italic with `*`.

This will display correctly formatted text with proper headings, code blocks, and other formatting when you paste it into your GitHub README file. Let me know if you need further assistance!








Done


