Getting Started: Create and Manage Cloud Resources: Challenge Lab

The steps to solve Create and Manage Cloud Resources: Challenge Lab on the Google Cloud Platform (Qwiklabs)


Topics tested:
Create an instance.
Create a 3 node Kubernetes cluster and run a simple service.
Create an HTTP(s) Load Balancer in front of two web servers.
Setup
Before you click the Start Lab button
Read these instructions. Labs are timed and you cannot pause them. The timer, which starts when you click Start Lab, shows how long Google Cloud resources will be made available to you.

This Qwiklabs hands-on lab lets you do the lab activities yourself in a real cloud environment, not in a simulation or demo environment. It does so by giving you new, temporary credentials that you use to sign in and access Google Cloud for the duration of the lab.

What you need
To complete this lab, you need:

Access to a standard internet browser (Chrome browser recommended).
Time to complete the lab.
Note: If you already have your own personal Google Cloud account or project, do not use it for this lab.

Note: If you are using a Pixelbook, open an Incognito window to run this lab.

Challenge scenario
You have started a new role as a Junior Cloud Engineer for Jooli Inc. You are expected to help manage the infrastructure at Jooli. Common tasks include provisioning resources for projects.

You are expected to have the skills and knowledge for these tasks, so don't expect step-by-step guides to be provided.

Some Jooli Inc. standards you should follow:

Create all resources in the default region or zone, unless otherwise directed.
Naming is normally team-resource, e.g. an instance could be named nucleus-webserver1
Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination 
(and possibly yours), so beware. This is the guidance the monitoring team is willing to share; unless directed use f1-micro for 
small Linux VMs and n1-standard-1 for Windows or other applications such as Kubernetes nodes.

Task 1: Create a project jumphost instance
Make sure you:

name the instance nucleus-jumphost
use the machine type f1-micro
use the default image type (Debian Linux)

In the Cloud Console, on the top left of the screen, select Navigation menu > Compute Engine > VM Instances:

Name for the VM instance : nucleus-jumphost

Region : leave Default Region

Zone : leave Default Zone

Machine Type : f1-micro (Series - N1)

Boot Disk : use the default image type (Debian Linux)

Create.

Click Check my progress to verify the objective.
Task Completed...
Task 2: Create a Kubernetes service cluster
The team is building an application that will use a service. This service will run on Kubernetes.

You need to:

Create a cluster (in the us-east1 region) to host the service
Use the Docker container hello-app (gcr.io/google-samples/hello-app:2.0) as a place holder, the team will replace the container with their own work later
Expose the app on port 8080
Activate Cloud Shell

0.set up cloud shell 

gcloud config set project qwiklabs-gcp-00-6da3aa6c7ad7

gcloud config set compute/zone us-west1-a

gcloud container clusters create nucleus-webserver1

gcloud container clusters get-credentials nucleus-webserver1

kubectl create deployment hello-app --image=gcr.io/google-samples/hello-app:2.0

kubectl expose deployment hello-app --type=LoadBalancer --port 8080

kubectl get service


Task 3: Setup an HTTP load balancer
We will serve the site via nginx web servers, but we want to ensure we have a fault tolerant environment, so please create an HTTP load balancer with a managed instance group of two nginx web servers. Use the following to configure the web servers, the team will replace this with their own configuration later.

You need to:

Create an instance template
Create a target pool
Create a managed instance group
Create a firewall rule to allow traffic (80/tcp)
Create a health check
Create a backend service and attach the manged instance group
Create a URL map and target HTTP proxy to route requests to your URL map
Create a forwarding rule

Perfrom the folloiwng in cloud shell to Setup an HTTP load balancer

0.Create startup.sh file

cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF

1 .Create an instance template :

gcloud compute instance-templates create nginx-template \
--metadata-from-file startup-script=startup.sh

2 .Create a target pool :

gcloud compute target-pools create nginx-pool

3 .Create a managed instance group :

gcloud compute instance-groups managed create nginx-group \
--base-instance-name nginx \
--size 2 \
--template nginx-template \
--target-pool nginx-pool

gcloud compute instances list

4 .Create a firewall rule to allow traffic (80/tcp) :

gcloud compute firewall-rules create grant-tcp-rule-888 --allow tcp:80

gcloud compute forwarding-rules create nginx-lb \
--region us-west1 \
--ports=80 \
--target-pool nginx-pool

gcloud compute forwarding-rules list


5 .Create a health check :

gcloud compute http-health-checks create http-basic-check
gcloud compute instance-groups managed set-named-ports nginx-group --named-ports http:80

6 .Create a backend service and attach the manged instance group :

gcloud compute backend-services create nginx-backend --protocol HTTP --http-health-checks http-basic-check --global
gcloud compute backend-services add-backend nginx-backend --instance-group nginx-group --instance-group-zone us-west1-a --global


7. Creating a URL map and target HTTP proxy to route requests to your URL map:
gcloud compute url-maps create web-map --default-service nginx-backend

gcloud compute target-http-proxies create http-lb-proxy --url-map web-map


8 .Create a forwarding rule :

gcloud compute forwarding-rules create http-content-rule --global --target-http-proxy http-lb-proxy --ports 80

gcloud compute forwarding-rules list
