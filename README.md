# Project #2: Highly Available Web Cluster & Autoscaling Application Load Balancer

## 📋 Real-World Operational Scenario
* **The Business Challenge:** Your production web application experiences highly variable web traffic spikes that threaten system availability. Corporate stability and budgeting mandates require an infrastructure configuration that automatically provisions extra server capacity during high CPU utilization windows and shrinks back down during idle frames to minimize runtime expenses, all managed behind a single high-performance network gateway.
* **The Technical Resolution:** Activated the Compute Engine subsystem and established a custom-mode network matrix (`practice-vpc` and `central-subnet`). Designed an un-immutable Instance Template running an automated Apache web server bootstrap script. Deployed a Managed Instance Group (MIG) governed by a 60% dynamic CPU Autoscaling policy, and anchored the entire cluster behind an Layer 7 Global External HTTP Application Load Balancer configuration.

## ⚡ Flattened One-Liner Execution Commands

```bash
# 1. Establish project environment target
export MY_PROJ="project-c1a05de0-ba4b-4764-93a"

# 2. Enable foundational Compute Engine resource management APIs
gcloud services enable compute.googleapis.com --project=\$MY_PROJ

# 3. Provision the custom VPC network base and regional subnet block
gcloud compute networks create practice-vpc --subnet-mode=custom --project=\$MY_PROJ
gcloud compute networks subnets create central-subnet --network=practice-vpc --region=us-central1 --range=10.0.1.0/24 --project=\$MY_PROJ

# 4. Generate the baseline instance template equipped with apache2 startup scripts
gcloud compute instance-templates create ace-web-template --machine-type=e2-micro --network=practice-vpc --subnet=central-subnet --tags=allow-http-traffic --metadata=startup-script='#!/bin/bash apt-get update && apt-get install -y apache2 echo "Welcome to Project 2 from \$(hostname)" > /var/www/html/index.html' --project=\$MY_PROJ

# 5. Initialize the Managed Instance Group (MIG) container framework
gcloud compute instance-groups managed create ace-web-mig --template=ace-web-template --size=2 --zone=us-central1-a --project=\$MY_PROJ

# 6. Apply the 60% CPU target threshold autoscaling policy constraints
gcloud compute instance-groups managed set-autoscaling ace-web-mig --max-num-replicas=5 --target-cpu-utilization=0.60 --cool-down-period=60 --zone=us-central1-a --project=\$MY_PROJ

# 7. Configure the global load balancing health checks and backend definitions
gcloud compute health-checks create http ace-http-health-check --port=80 --project=\$MY_PROJ
gcloud compute backend-services create ace-backend-service --protocol=HTTP --health-checks=ace-http-health-check --global --project=\$MY_PROJ
gcloud compute backend-services add-backend ace-backend-service --instance-group=ace-web-mig --instance-group-zone=us-central1-a --global --project=\$MY_PROJ

# 8. Finalize the URL routing maps, proxies, and edge forwarding rule pipelines
gcloud compute url-maps create ace-web-map --default-service=ace-backend-service --project=\$MY_PROJ
gcloud compute target-http-proxies create ace-http-proxy --url-map=ace-web-map --project=\$MY_PROJ
gcloud compute forwarding-rules create ace-http-forwarding-rule --global --target-http-proxy=ace-http-proxy --ports=80 --project=\$MY_PROJ
```

## 🔍 Validation Protocol
Confirm that your load balancer's forwarding rule network endpoint is actively listening for public HTTP connections:
```bash
gcloud compute forwarding-rules list --global --project=\$MY_PROJ
```
