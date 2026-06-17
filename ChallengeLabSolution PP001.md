Create a Landing Zone and Harden it with Security Command Center: Challenge Lab
<br><br>
gcloud config set project $HOST_PROJECT_ID

Task 1:<br><br>
gcloud config set project qwiklabs-gcp-04-15007f6e147f
mkdir shared-vpc
gsutil -m cp -r gs://qwiklabs-gcp-04-15007f6e147f-labconfig-bucket/*  ~/shared-vpc/
cd shared-vpc/
cat <<EOF > variable.tf
variable "gcp_project_id" {
  type        = string
  description = "The GCP Project ID to apply this config to."
  default = ""
}
EOF
terraform init
terraform plan
terraform apply -auto-approve
export HOST_PROJECT_ID="qwiklabs-gcp-04-15007f6e147f"
export SERVICE_PROJECT_ID="qwiklabs-gcp-01-fdf2db5dda3c"
gcloud compute shared-vpc enable $HOST_PROJECT_ID
gcloud compute shared-vpc associated-projects add $SERVICE_PROJECT_ID --host-project=$HOST_PROJECT_ID
export HOST_PROJECT_NUM=$(gcloud projects describe $HOST_PROJECT_ID --format="value(projectNumber)")

export SERVICE_PROJECT_NUM=$(gcloud projects describe $SERVICE_PROJECT_ID --format="value(projectNumber)")
gcloud compute subnets add-iam-policy-binding shared-network-subnet-01 \
    --project=$HOST_PROJECT_ID \
    --region=$LAB_REGION \
    --role="roles/compute.networkUser" \
    --member="serviceAccount:$SERVICE_PROJECT_NUM@://gserviceaccount.com"
    
gcloud compute subnets add-iam-policy-binding shared-network-subnet-01 \
    --project=$HOST_PROJECT_ID \
    --region=$LAB_REGION \
    --role="roles/compute.networkUser" \
    --member="serviceAccount:service-$SERVICE_PROJECT_NUM@://gserviceaccount.com"
    
gcloud compute subnets add-iam-policy-binding shared-network-subnet-02 \
    --project=$HOST_PROJECT_ID \
    --region=$LAB_REGION \
    --role="roles/compute.networkUser" \
    --member="serviceAccount:$SERVICE_PROJECT_NUM@://gserviceaccount.com"
    
gcloud compute subnets add-iam-policy-binding shared-network-subnet-02 \
    --project=$HOST_PROJECT_ID \
    --region=$LAB_REGION \
    --role="roles/compute.networkUser" \
    --member="serviceAccount:service-$SERVICE_PROJECT_NUM@://gserviceaccount.com"

    
Task 2:    

gcloud compute networks subnets update shared-network-subnet-01 \
    --region=us-central1 \
    --enable-private-ip-google-access

gcloud compute networks subnets update shared-network-subnet-02 \
    --region=us-central1 \
    --enable-private-ip-google-access
    
gcloud compute firewall-rules create cymbal-firewall-rule \
    --network=shared-network \
    --direction=INGRESS \
    --priority=1000 \
    --action=ALLOW \
    --rules=tcp:22 \
    --source-ranges=35.235.240.0/20

 gcloud compute routers create landing-zone-router\
    --network=shared-network \
    --region=us-central1
  
 gcloud compute routers nats create cymbal-gateway \
    --router=landing-zone-router\
    --region=us-central1 \
    --auto-allocate-nat-external-ips \
    --nat-all-subnet-ip-ranges
 
Task 3:

cd
git clone https://github.com/terraform-google-modules/terraform-google-vm.git

# Navigate to the target example folder
cd terraform-google-vm/examples/compute_instance/simple

sed -i 's|subnetwork\s*=\s*".*"|subnetwork = "projects/qwiklabs-gcp-01-815ac148bbc7/regions/us-central1/subnetworks/shared-network-subnet-01"|g' main.tf
    

terraform init
terraform plan
terraform apply -auto-approve    
 
gsutil -m cp -r gs://qwiklabs-gcp-03-f86c00bdc9cf-labconfig-service-project/*  ~/compute-address/ 
 
 
 ----------------------   
    
    cat <<EOF > variables.tf
variable "project_id" {
  description = "The GCP project ID"
  type        = string
  default     = "qwiklabs-gcp-02-53a3ef132491"
}




variable "region" {
  description = "The GCP region to deploy instances into"
  type        = string
  default     = "us-central1"
}

variable "subnetwork" {
  description = "The subnetwork to deploy instances into"
  type        = string
  default     = "projects/qwiklabs-gcp-02-53a3ef132491/regions/us-central1/subnetworks/shared-network-subnet-01"
}
EOF

# Build a hardened main.tf file configuration
cat <<EOF > main.tf
provider "google" {
  project = var.project_id
  region  = var.region
}

module "compute_instance" {
  source       = "../../../modules/compute_instance"
  project_id   = var.project_id
  zone         = "us-central1-a"
  hostname     = "cymbal-service-instance"
  subnetwork   = var.subnetwork
  num_instances = 1

  # Security Hardening parameters
  access_config   = [] # Empty array ensures NO external IP assignment
  metadata = {
    serial-port-enable = "FALSE"
  }
}
EOF
