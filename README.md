# GADS-2020-PhaseII-ProjectExercises
### Submissions of Phase II GADS Cloud Programme.
<details>
<summary>Lab 1: Getting Started with Cloud Marketplace</summary>
  <img src="Screenshots/Lab 1 Getting Started with Cloud Marketplace.png">
</details>
 
 <details>
<summary>Lab 2: Google Cloud Fundamentals - Getting Started with Compute Engine</summary>
  <img src="Screenshots/Lab 2 Google Cloud Fundamentals - Getting Started with Compute Engine.png">
</details>
  
  <details>
<summary>Lab 3: Getting Started with Cloud Storage and Cloud SQL</summary>
  <img src="Screenshots/Lab 3 Getting Started with Cloud Storage and Cloud SQL.png">
</details>
  
  <details>
<summary>Lab 4: Getting Started with GKE</summary>
  <img src="Screenshots/Lab 4 Getting Started with GKE.png">
</details>
  
  <details>
<summary>Lab 5: Getting Started with App Engine</summary>
  <img src="Screenshots/Lab 5 Getting Started with App Engine.png">
</details>
  
  <details>
<summary>Lab 6: Getting Started with Deployment Manager and Cloud Monitoring</summary>
  <img src="Screenshots/Lab 6 Getting Started with Deployment Manager and Cloud Monitoring.png">
</details>
  
  <details>
<summary>Lab 7: Getting Started with BigQuery</summary>
  <img src="Screenshots/Lab 7 Getting Started with BigQuery.png">
</details>
  
  <details>
<summary>Lab 8: Infrastructure Preview</summary>
  <img src="Screenshots/Lab 8 Infrastructure Preview.png">
</details>
  
  <details>
<summary>Lab 9: VPC Networking</summary>
  <img src="Screenshots/Lab 9 VPC Networking.png">
</details>

<details>
<summary>Lab 9: Cloud IAM </summary>
<img src="Screenshots/Lab 10 Cloud IAM.png">
</details>
  

<details>
<summary>## Translation Code 1: Lab 11 Configuring an HTTP Load Balancer with Autoscaling </summary>

1. Configure HTTP and health check firewall rules:
```
gcloud compute --project=qwiklabs-gcp-508906201563c6b8 firewall-rules create fw-allow-health-checks --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=130.211.0.0/22,35.191.0.0/16 --target-tags=allow-health-checks
```

2. Create a NAT configuration using Cloud Router:
```
gcloud compute routers nats create nat-config \
    --router=nat-config \
    --nat-all-subnet-ip-ranges \
    --region=us-central1
```

3. Create a custom image for a web server:
```
gcloud beta compute --project=qwiklabs-gcp-508906201563c6b8 instances create webserver --zone=us-central1-a --machine-type=f1-micro --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --tags=allow-health-checks --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --no-boot-disk-auto-delete --boot-disk-type=pd-standard --boot-disk-device-name=webserver --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any
```

4. Configure Apache2 via ssh:
```
sudo apt-get update
sudo apt-get install -y apache2

sudo service apache2 start

sudo update-rc.d apache2 enable
```

5. Create a custom image for disk:
```
gcloud compute images create mywebserver --project=qwiklabs-gcp-508906201563c6b8 --source-disk=webserver --source-disk-zone=us-central1-a --storage-location=us
```

6. Configure an instance template and create instance groups:
```
Configure Instance Template:
gcloud beta compute --project=qwiklabs-gcp-508906201563c6b8 instance-templates create mywebserver-template --machine-type=f1-micro --network=projects/qwiklabs-gcp-508906201563c6b8/global/networks/default --no-address --maintenance-policy=MIGRATE --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=mywebserver-template --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
```
6.1 Configure Instant Group for us-central1-a:
```
gcloud compute --project "qwiklabs-gcp-508906201563c6b8" health-checks create tcp "http-health-check" --timeout "5" --check-interval "10" --unhealthy-threshold "3" --healthy-threshold "2" --port "80"

gcloud beta compute --project=qwiklabs-gcp-508906201563c6b8 instance-groups managed create us-central1-mig --base-instance-name=us-central1-mig --template=mywebserver-template --size=1 --zones=us-central1-b,us-central1-c,us-central1-f --instance-redistribution-type=PROACTIVE --health-check=http-health-check --initial-delay=60

gcloud beta compute --project "qwiklabs-gcp-508906201563c6b8" instance-groups managed set-autoscaling "us-central1-mig" --region "us-central1" --cool-down-period "60" --max-num-replicas "2" --min-num-replicas "1" --target-load-balancing-utilization "0.8" --mode "on"
```

6.2 Configure Instant Group for europewest1:
```
gcloud beta compute --project=qwiklabs-gcp-508906201563c6b8 instance-groups managed create europe-west1-mig --base-instance-name=europe-west1-mig --template=mywebserver-template --size=1 --zones=europe-west1-b,europe-west1-c,europe-west1-d --instance-redistribution-type=PROACTIVE --health-check=http-health-check --initial-delay=60

gcloud beta compute --project "qwiklabs-gcp-508906201563c6b8" instance-groups managed set-autoscaling "europe-west1-mig" --region "europe-west1" --cool-down-period "60" --max-num-replicas "2" --min-num-replicas "1" --target-load-balancing-utilization "0.8" --mode "on"
```

7. Configure the HTTP load balancer:
   ```
   gcloud compute health-checks create http http-basic-check \
        --port 80
    
    gcloud compute backends-services create http-backend \
        --protocol=HTTP \
        --port-name=http \
        --health-checks=http-basic-check \
        --global

    gcloud compute backends-services add-backend http-backends \
        --instance-group=lb-backend-example \
        --instance-group-zone=europe-west1-mig \
	      --instance-group-zone=us-central1-mig \
        --global
    ```
    

8. Stress test the HTTP load balancer:

8.1 Create a stress test VM:
```
gcloud beta compute --project=qwiklabs-gcp-508906201563c6b8 instances create stress-test --zone=us-west1-c --machine-type=f1-micro --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=stress-test --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
```

8.2 ssh from stress-test instance:
```
export LB_IP=34.120.183.99

ab -n 500000 -c 1000 http://$LB_IP/
```
</detail>


<details>
<summary>## Translation Code 2: Lab 12 </summary>




</detail>
