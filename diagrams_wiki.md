# Diagrams (mingrammer/diagrams) Guide

## Installation

```bash
pip install diagrams
# Diagrams requires Graphviz to be installed
# On Ubuntu/Debian:
sudo apt-get install graphviz
# On macOS:
brew install graphviz
# On Windows: Download from https://graphviz.org/download/
```

## Overview

**Specialty:** Cloud architecture and system diagrams

**Best Use Cases:**
- Cloud infrastructure architecture diagrams
- System design documentation
- DevOps pipeline visualization
- Microservices architecture diagrams
- Infrastructure as Code documentation
- Technical presentations and proposals

## ASCII Example

Below is an ASCII representation of what we'll create - a comprehensive cloud architecture diagram:

```
                    ┌─────────────┐
                    │   Users     │
                    └──────┬──────┘
                           │
    ┌──────────────────────▼──────────────────────┐
    │                AWS Cloud                    │
    │  ┌─────────┐  ┌─────────┐  ┌─────────┐     │
    │  │   WAF   │─▶│   ALB   │─▶│   ECS   │     │
    │  └─────────┘  └─────────┘  └────┬────┘     │
    │                                 │          │
    │  ┌─────────┐  ┌─────────┐  ┌────▼────┐     │
    │  │   S3    │  │   RDS   │  │ Lambda  │     │
    │  └─────────┘  └─────────┘  └─────────┘     │
    └─────────────────────────────────────────────┘
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
    ┌────▼────┐       ┌────▼────┐       ┌────▼────┐
    │   GCP   │       │  Azure  │       │ On-Prem │
    │   GKE   │       │   AKS   │       │Monitoring│
    └─────────┘       └─────────┘       └─────────┘

Multi-Cloud Infrastructure with:
- AWS: Primary cloud with ECS, Lambda, RDS, S3
- GCP: Backup region with Kubernetes
- Azure: Edge locations and CDN
- Security: WAF, DDoS protection, IAM
- Monitoring: Cross-cloud observability
```

## Step-by-Step Tutorial

### 1. Setup and Imports

```python
from diagrams import Diagram, Cluster, Edge
from diagrams.aws.compute import EC2, ECS, Lambda
from diagrams.aws.database import RDS, ElastiCache
from diagrams.aws.network import ELB, CloudFront, Route53
from diagrams.aws.storage import S3
from diagrams.aws.security import IAM
from diagrams.onprem.client import Users
```

### 2. Prepare Your Data

```python
# Define the architecture components
# This will be represented through the diagram structure
# No separate data preparation needed for diagrams
```

### 3. Create the Basic Visualization

```python
# Create a basic web application architecture
with Diagram("Web Application Architecture", show=False, direction="TB"):
    users = Users("Users")

    with Cluster("AWS Cloud"):
        dns = Route53("DNS")
        cdn = CloudFront("CDN")

        with Cluster("Load Balancing"):
            lb = ELB("Load Balancer")

        with Cluster("Web Tier"):
            web_servers = [
                EC2("Web Server 1"),
                EC2("Web Server 2"),
                EC2("Web Server 3")
            ]

        with Cluster("Database Tier"):
            primary_db = RDS("Primary DB")
            cache = ElastiCache("Cache")

    # Define connections
    users >> dns >> cdn >> lb
    lb >> web_servers
    web_servers >> primary_db
    web_servers >> cache
```

### 4. Customize the Appearance

```python
# Create an enhanced microservices architecture
with Diagram("Microservices Architecture",
             filename="microservices_arch",
             show=False,
             direction="TB",
             graph_attr={"bgcolor": "white", "pad": "0.5"}):

    users = Users("Mobile & Web Users")

    with Cluster("API Gateway Layer"):
        api_gateway = ELB("API Gateway")

    with Cluster("Authentication"):
        auth_service = Lambda("Auth Service")
        auth_db = RDS("User DB")

    with Cluster("Core Services"):
        with Cluster("User Service"):
            user_api = ECS("User API")
            user_cache = ElastiCache("User Cache")

        with Cluster("Product Service"):
            product_api = ECS("Product API")
            product_db = RDS("Product DB")

        with Cluster("Order Service"):
            order_api = ECS("Order API")
            order_db = RDS("Order DB")
            order_queue = Lambda("Order Queue")

    with Cluster("Storage"):
        file_storage = S3("File Storage")

    with Cluster("Security"):
        iam = IAM("IAM Roles")

    # Define enhanced connections with labels
    users >> Edge(label="HTTPS") >> api_gateway
    api_gateway >> Edge(label="Auth") >> auth_service
    auth_service >> auth_db

    api_gateway >> Edge(label="User Requests") >> user_api
    user_api >> user_cache

    api_gateway >> Edge(label="Product Requests") >> product_api
    product_api >> product_db

    api_gateway >> Edge(label="Order Requests") >> order_api
    order_api >> order_db
    order_api >> Edge(label="Async Processing") >> order_queue

    [user_api, product_api, order_api] >> file_storage
    iam >> [auth_service, user_api, product_api, order_api]
```

### 5. Create Advanced Cloud Infrastructure Diagrams

```python
# Create a comprehensive multi-cloud architecture
with Diagram("Multi-Cloud Architecture",
             filename="multi_cloud_arch",
             show=False,
             direction="TB"):

    from diagrams.aws.compute import EC2, ECS, Lambda
    from diagrams.aws.database import RDS, Dynamodb
    from diagrams.aws.network import VPC, PrivateSubnet, PublicSubnet, InternetGateway
    from diagrams.aws.storage import S3
    from diagrams.aws.security import WAF, Shield
    from diagrams.gcp.compute import GKE
    from diagrams.azure.compute import AKS
    from diagrams.onprem.monitoring import Prometheus, Grafana

    users = Users("Global Users")

    with Cluster("AWS Region - US East"):
        with Cluster("VPC"):
            igw = InternetGateway("Internet Gateway")

            with Cluster("Public Subnet"):
                waf = WAF("Web Application Firewall")
                shield = Shield("DDoS Protection")
                alb = ELB("Application Load Balancer")

            with Cluster("Private Subnet - Web Tier"):
                web_servers = [
                    ECS("Web Service 1"),
                    ECS("Web Service 2"),
                    ECS("Web Service 3")
                ]

            with Cluster("Private Subnet - App Tier"):
                app_servers = [
                    Lambda("Auth Service"),
                    Lambda("Payment Service"),
                    Lambda("Notification Service")
                ]

            with Cluster("Database Tier"):
                primary_db = RDS("Primary Database")
                cache_db = Dynamodb("Session Cache")
                file_storage = S3("File Storage")

    with Cluster("Google Cloud - Europe"):
        gcp_cluster = GKE("GKE Cluster")

    with Cluster("Azure - Asia Pacific"):
        azure_cluster = AKS("AKS Cluster")

    with Cluster("Monitoring & Observability"):
        monitoring = Prometheus("Prometheus")
        dashboard = Grafana("Grafana Dashboard")

    # Define connections
    users >> waf >> shield >> alb
    alb >> web_servers
    web_servers >> app_servers
    app_servers >> [primary_db, cache_db, file_storage]

    # Multi-cloud connections
    web_servers >> Edge(label="Backup Region") >> gcp_cluster
    web_servers >> Edge(label="CDN Edge") >> azure_cluster

    # Monitoring connections
    [web_servers, app_servers] >> monitoring >> dashboard

# Create a Kubernetes infrastructure diagram
with Diagram("Kubernetes Infrastructure",
             filename="k8s_infrastructure",
             show=False,
             direction="TB"):

    from diagrams.k8s.compute import Pod, ReplicaSet, Deployment
    from diagrams.k8s.network import Service, Ingress
    from diagrams.k8s.storage import PV, PVC, StorageClass
    from diagrams.k8s.rbac import ServiceAccount
    from diagrams.k8s.others import HPA

    users = Users("External Users")

    with Cluster("Kubernetes Cluster"):
        ingress = Ingress("Nginx Ingress")

        with Cluster("Frontend Namespace"):
            frontend_svc = Service("Frontend Service")
            frontend_deploy = Deployment("Frontend Deployment")
            frontend_pods = [
                Pod("Frontend Pod 1"),
                Pod("Frontend Pod 2"),
                Pod("Frontend Pod 3")
            ]
            frontend_hpa = HPA("Frontend HPA")

        with Cluster("Backend Namespace"):
            backend_svc = Service("Backend Service")
            backend_deploy = Deployment("Backend Deployment")
            backend_pods = [
                Pod("Backend Pod 1"),
                Pod("Backend Pod 2")
            ]
            backend_sa = ServiceAccount("Backend SA")

        with Cluster("Database Namespace"):
            db_svc = Service("Database Service")
            db_deploy = Deployment("Database Deployment")
            db_pod = Pod("Database Pod")

            # Storage
            storage_class = StorageClass("SSD Storage")
            pv = PV("Persistent Volume")
            pvc = PVC("Database PVC")

    # Define connections
    users >> ingress >> frontend_svc >> frontend_deploy >> frontend_pods
    frontend_pods >> backend_svc >> backend_deploy >> backend_pods
    backend_pods >> db_svc >> db_deploy >> db_pod

    # Storage connections
    storage_class >> pv >> pvc >> db_pod

    # Auto-scaling
    frontend_hpa >> frontend_deploy

# Create a serverless architecture
with Diagram("Serverless Architecture",
             filename="serverless_arch",
             show=False,
             direction="LR"):

    from diagrams.aws.compute import Lambda
    from diagrams.aws.integration import SQS, SNS, Eventbridge
    from diagrams.aws.database import Dynamodb
    from diagrams.aws.storage import S3
    from diagrams.aws.network import APIGateway, CloudFront
    from diagrams.aws.analytics import Kinesis

    users = Users("Mobile/Web Users")

    with Cluster("API Layer"):
        api_gw = APIGateway("API Gateway")
        cdn = CloudFront("CloudFront CDN")

    with Cluster("Compute Layer"):
        auth_lambda = Lambda("Authentication")
        user_lambda = Lambda("User Management")
        order_lambda = Lambda("Order Processing")
        payment_lambda = Lambda("Payment Processing")
        notification_lambda = Lambda("Notifications")

    with Cluster("Event Processing"):
        event_bus = Eventbridge("Event Bus")
        queue = SQS("Processing Queue")
        topic = SNS("Notification Topic")
        stream = Kinesis("Data Stream")

    with Cluster("Storage Layer"):
        user_db = Dynamodb("User Database")
        order_db = Dynamodb("Order Database")
        files = S3("File Storage")
        analytics_db = Dynamodb("Analytics Database")

    # Define serverless flow
    users >> cdn >> api_gw
    api_gw >> [auth_lambda, user_lambda, order_lambda]

    # Event-driven architecture
    order_lambda >> event_bus >> [payment_lambda, notification_lambda]
    payment_lambda >> queue >> notification_lambda
    notification_lambda >> topic

    # Data flow
    [user_lambda, order_lambda] >> stream >> analytics_db
    user_lambda >> user_db
    order_lambda >> order_db
    [auth_lambda, user_lambda] >> files
```

### 6. Export or Display

```python
# The diagrams are automatically saved when the context manager exits
# Files are saved as PNG by default

# To save in different formats, you can specify:
with Diagram("Architecture Overview",
             filename="overview",
             show=False,
             outformat="svg"):  # Can be 'png', 'svg', 'pdf', 'dot'

    # Your diagram code here
    users = Users("Users")
    app = EC2("Application")
    db = RDS("Database")

    users >> app >> db

print("Cloud Infrastructure Diagrams saved successfully!")
print("Files generated:")
print("- web_application_architecture.png")
print("- microservices_arch.png")
print("- multi_cloud_arch.png")
print("- k8s_infrastructure.png")
print("- serverless_arch.png")
print("- overview.svg")

print("\nCloud Infrastructure Examples Created:")
print("✓ Multi-cloud architecture (AWS + GCP + Azure)")
print("✓ Kubernetes infrastructure with namespaces")
print("✓ Serverless event-driven architecture")
print("✓ Microservices with security and monitoring")
print("✓ Traditional web application architecture")
```

## Common Pitfalls and Solutions

- **Pitfall 1**: Missing Graphviz installation
  - Solution: Install Graphviz system package before using diagrams
- **Pitfall 2**: Icons not displaying correctly
  - Solution: Ensure you're using the correct provider icons (aws, gcp, azure, etc.)
- **Pitfall 3**: Complex layouts becoming cluttered
  - Solution: Use clusters effectively and consider breaking into multiple diagrams

## Advanced Techniques

- Creating custom icons and providers for specialized systems
- Using different cloud providers (AWS, GCP, Azure) in the same diagram
- Implementing conditional diagram generation based on configuration
- Creating diagram templates for common architecture patterns
- Integrating with infrastructure as code tools (Terraform, CloudFormation)
- Generating diagrams from existing infrastructure discovery

## Resources

- [Diagrams Documentation](https://diagrams.mingrammer.com/)
- [Available Icons and Providers](https://diagrams.mingrammer.com/docs/nodes/aws)
- [Diagrams Examples](https://diagrams.mingrammer.com/docs/getting-started/examples)
- [Architecture Diagram Best Practices](https://aws.amazon.com/architecture/)
