1.Create Reserved External Static IP
gcloud compute addresses create cepf-static-ip --global
gcloud compute addresses describe cepf-static-ip --global --format="value(address)"

2.Create Classic certificate [google Managed]
gcloud compute ssl-certificates create webserver-cert --description=WebserverCertificate --domains=8-232-98-86.qlencrypt.com --global

gcloud compute instance-groups unmanaged create webserver-instance-group \
    --zone=us-west1-c
gcloud compute instance-groups unmanaged add-instances webserver-instance-group \
    --instances=webserver1,webserver2 \
    --zone=us-west1-c

gcloud compute health-checks create http http-basic-check --port 80
gcloud compute firewall-rules create allow-gfe-health-checks \
    --direction=INGRESS \
    --action=ALLOW \
    --rules=tcp:80 \
    --source-ranges=35.191.0.0/16,130.211.0.0/22 


gcloud compute backend-services create cepf-webserver-backend \
    --load-balancing-scheme=EXTERNAL_MANAGED \
    --protocol=HTTP \
    --port-name=http \
    --health-checks=http-basic-check \
    --global

gcloud compute backend-services add-backend cepf-webserver-backend --instance-group=webserver-instance-group --instance-group-zone=us-west1-c --global

gcloud compute url-maps create webserver-map \
    --default-service=cepf-webserver-backend

gcloud compute target-https-proxies create https-lb-proxy \
    --url-map=webserver-map \
    --ssl-certificates=webserver-cert
    
gcloud compute forwarding-rules create https-forwarding-rule \
    --load-balancing-scheme=EXTERNAL_MANAGED \
    --network-tier=PREMIUM \
    --address=cepf-static-ip \
    --ports=443 \
    --target-https-proxy=https-lb-proxy \
    --global
  
cat <<EOF > redirect.yaml
name: redirect-map
defaultUrlRedirect:
  redirectResponseCode: MOVED_PERMANENTLY_DEFAULT
  httpsRedirect: true
EOF

gcloud compute url-maps import redirect-map \
    --source=redirect.yaml \
    --global
    
