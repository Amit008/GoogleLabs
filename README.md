Task 1. Set up an SSL certificate resource
Set up a global static external IP address named cepf-static-ip that will be used to reach the load balancer.

For an HTTPS load balancer, create an SSL certificate resource with the following command:

gcloud compute ssl-certificates create webserver-cert \
    --description=WebserverCertificate \
    --domains=00-00-00-00.qlencrypt.com \
    --global
Replace 00-00-00-00 above with the global static external IP address you created in the same format (with each octet separated by dashes - NOT DECIMALS).
Create an SSL certificate.<br> <br>
Task 2. Create an unmanaged instance group
Cymbal's application teams have deployed a web application on VM instances called webserver1 and webserver2. You have to make this web applications available to the public internet via a highly available Global external Application Load Balancing service.
Create an unmanaged instance group webserver-instance-group in the default zone zone.
Create an un-managed instance group.
Add pre-created instances webserver1 and webserver2 to the unmanaged instance group.
Add instances to the instance group.<br> <br>
Task 3. Set up the load balancer
Set up a Global external Application Load Balancer with the instance group backend you created in previous step.
Name the backend service in the load-balancer cepf-webserver-backend.
Enable HTTP to HTTPS Redirect.Verify that the web application can be accessed via the Load balancer static IP.
