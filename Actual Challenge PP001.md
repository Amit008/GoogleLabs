
Task 1. Configure the Shared VPC and service project
In this task, you will set up a Shared VPC in Google Cloud using Terraform Cloud Foundation Fabric (CFF) modules that have been copied to a Cloud Storage bucket named Bucket Name.

You will create a VPC network named Network Name in the designated Host Project (Host Project ID) and configure a Service Project (Service Project ID) to use a shared subnet from the host project.

Note: Ensure that the network configuration avoids overlapping with the on-premises 10.0.0.0/8 CIDR range used by Cymbal Bank.
In Host Project Cloud Shell, create the shared-vpc folder.

To create the shared VPC network, copy all files from the cloud storage bucket named Bucket Name to the shared-vpc folder:

gsutil -m cp -r gs://Bucket Name/*  ~/shared-vpc/
Copied!
In the Cloud Shell Editor, navigate to the shared-vpc folder and review the files.

Update the variables.tf file with the following values:

Variable Name	Value
Host Project ID	Host Project ID
Region	Lab Region
Next, use Terraform to deploy the Shared VPC network in the Host Project.

The VPC network named Network Name includes two subnets with the following configurations:

Subnet Name	CIDR Range
shared-network-subnet-01	172.16.10.0/24
shared-network-subnet-02	172.16.20.0/24
Using the Google Cloud Console, enable Shared VPC on the Host Project (Host Project ID).

Attach the Service Project (Service Project ID) to the Host Project and assign all required roles.

Configure the Host Project to grant the Service Project access to the shared-network-subnet-01 and shared-network-subnet-02 subnets.

Click Check my progress to verify the objective
Configure the Shared VPC and service project

Task 2. Secure the network and enforce policies
In this task, you will configure networking and security controls in a Shared VPC environment to ensure secure access to Google services and virtual machines.

Enable Private Google Access on the two Shared VPC subnets, shared-network-subnet-01 and shared-network-subnet-02, in the Host Project to allow secure internal communication with Google services.

Create a firewall rule named Firewall Rule Name to allow SSH access only from the IAP IP range 35.235.240.0/20 using the VPC network Network Name.

This ensures that even if a VM obtains an external IP, the firewall prevents direct internet access. For clients, you will use an Organizational Policy to enforce this rule.

Click Check my progress to verify the objective
Secure the network and enforce policies

Enable Outbound Internet Access for Private VM via Cloud NAT in Host Project
Configure Cloud NAT in the Host Project to allow private VM in the Service project (using Shared VPC) to access the internet for updates without assigning external IPs to the VMs.

Create a Cloud NAT gateway in the Host Project using the following configurations:

Parameter	Value
Gateway Name	cymbal-gateway
Network	Network Name
Region	Lab Region
Cloud Router Name	Cloud Router Name
Source subnet	Primary and secondary ranges for all subnets
Click Check my progress to verify the objective
Enable Outbound Internet Access for Private VM

Task 3. Deploy a Secure VM Instance and Reserve an Internal Static IP
In this task, you deploy a standard VM instance named Instance Name in the Lab Region region using the provided Cloud Foundation module in the Service Project. You will reserve an internal static IP by using the code stored in the Cloud Storage bucket named Service Project Bucket Name.

In the Service Project, navigate to the Cloud Shell and clone the Cloud Foundation repo.

https://github.com/terraform-google-modules/terraform-google-vm.git
Copied!
In Cloud Shell, navigate to the following folder: terraform-google-vm/examples/compute_instance/simple and update the variables.tf file with region Lab Region. Also update main.tf file with the Hostname value with Instance Name.

You must configure the instance to meet the following security requirements:

No External IP: The network interface must not be assigned a public IP address.
Serial Port Disabled: The instance metadata must explicitly set serial-port-enable to FALSE.
After updating the security requirements, use Terraform to deploy a secure VM instance in the Service Project.

Use the subnetwork link as Subnetwork Link.
In the Service Project Cloud Shell, create a folder named compute-address and copy all files from the Cloud Storage bucket Service Project Bucket Name into it.

gsutil -m cp -r gs://Service Project Bucket Name/*  ~/compute-address/
Copied!
Navigate to the compute-address folder and update the variables.tf file to set the default value of the subnetwork variable to Subnetwork Link.

Use Terraform to create the reserved internal static IP named Static IP Name in the Service Project.

Click Check my progress to verify the objective.
Deploy a Secure VM Instance and Reserve an Internal Static IP

Task 4. Enable Security Command Center
In this task, you activate Security Command Center (Standard Tier) at the project level for both the Host and Service projects to establish centralized asset monitoring and visibility.

