# LAB: Google Cloud Fundamentals: Getting Started with GKE

## Objectives:

In this lab, you learn how to perform the following tasks:

    - Provision a Kubernetes cluster using Kubernetes Engine.

    - Deploy and manage Docker containers using kubectl.

## Steps:

1. Confirm that both of these APIs are enabled from the Cloud Console:

    - Kubernetes Engine API.
    - Container Registry API.


2. Start a Kubernetes Engine cluster using the gcloud command line interface.

  a. Place the zone assigned you to into an environment variable called MY_ZONE. At the Cloud Shell prompt, type this partial command:

    - export MY_ZONE=us-central1-a


  b. Start a Kubernetes cluster managed by Kubernetes Engine. Name the cluster webfrontend and configure it to run 2 nodes:

    - gcloud container clusters create webfrontend --zone $MY_ZONE --num-nodes 2


  c. After the cluster is created, check your installed version of Kubernetes using the kubectl version command:

    - kubectl version


  d. View your running nodes in the GCP Console on Compute Engine>VM instances.
  Your Kubernetes cluster is now ready for use.


3. Run and deploy a container.

  a. From your Cloud Shell prompt, launch a single instance of the nginx container:

    - kubectl create deploy nginx --image=nginx:1.17.10


  b. View the pod running the nginx container:

    - kubectl get pods


  c. Expose the nginx container to the Internet:

    - kubectl expose deployment nginx --port 80 --type LoadBalancer

      - Result:
        Kubernetes created a service and an external load balancer with a public IP address attached to it. The IP address remains the same for the life of the service.


  d. View the new service:

    - kubectl get services

      - Result:
        It may take a few seconds before the External-IP field is populated for your service. This is normal. Just re-run the kubectl get services command every few seconds until the field is populated.

You can use the displayed external IP address to test and contact the nginx container remotely.


  e. Open a new web browser tab and paste your cluster's external IP address into the address bar. The default home page of the Nginx browser is displayed.


  f. Scale up the number of pods running on your service:

    - kubectl scale deployment nginx --replicas 3


  g. Confirm that Kubernetes has updated the number of pods:

    - kubectl get pods


  h. Confirm that your external IP address has not changed:

    - kubectl get services


  i. Return to the web browser tab in which you viewed your cluster's external IP address. Refresh the page to confirm that the nginx web server is still responding.
