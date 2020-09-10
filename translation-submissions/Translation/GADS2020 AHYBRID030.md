# LAB: AHYBRID030 Installing the Istio on GKE Add-On with Kubernetes Engine

## Objectives:

In this lab, you learn how to perform the following tasks:

    - Provision a cluster on Google Kubernetes Engine (GKE)

    - Install and configure the Istio on GKE Add-On, which includes the Istio control-plane and a method to deploy Envoy proxies as sidecars

    - Deploy Bookinfo, an Istio-enabled multi-service application

    - Enable external access using an Istio Ingress Gateway

    - Use the Bookinfo application

## Steps:

Task 1. Install and configure a cluster with the Istio on GKE Add-On

In this lab, you'll use a Google Kubernetes Engine (GKE) cluster named central, in the us-central1 region.

In this task, you:

    - Deploy the central cluster on GKE using Cloud Shell.

    - Deploy a service mesh using the Istio on GKE add-on.


1. Use Cloud Shell to deploy your GKE cluster with Istio:

  a. Set environment variables for the zone and cluster name:

    - export CLUSTER_NAME=central
      export CLUSTER_ZONE=us-central1-b
      export CLUSTER_VERSION=latest


  b. Create your GKE cluster, using the Istio on GKE add-on:

    - gcloud beta container clusters create $CLUSTER_NAME --zone $CLUSTER_ZONE --num-nodes 4 --machine-type "n1-standard-2" --image-type "COS" --cluster-version=$CLUSTER_VERSION --enable-ip-alias --addons=Istio --istio-config=auth=MTLS_STRICT

      Result:
        This command creates a cluster in a single zone, with 4 nodes.


  c. Configure kubectl command line access by running:

     - export GCLOUD_PROJECT=$(gcloud config get-value project)

       gcloud container clusters get-credentials $CLUSTER_NAME --zone $CLUSTER_ZONE --project $GCLOUD_PROJECT


  d. Grant admin permissions in the cluster to the current gcloud user:

     - kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value core/account)


  e. Check that your cluster is up and running:

    - gcloud container clusters list

    Result:
      NAME     LOCATION       MASTER_VERSION  ... STATUS
      central  us-central1-b  1.16.9-gke.2        RUNNING


  f. Ensure the following Kubernetes services are deployed: istio-citadel, istio-galley, istio-pilot, istio-ingressgateway, istio-policy, istio-sidecar-injector, and istio-telemetry. You'll also see other deployed services:

    - kubectl get service -n istio-system


  g. Ensure the corresponding Kubernetes pods are deployed and all containers are up and running: istio-pilot, istio-galley, istio-policy, istio-telemetry, istio-ingressgateway, istio-sidecar-injector, and istio-citadel:

    - kubectl get pods -n istio-system

      Result:
        All instances should be running.



Task 2. Deploy Bookinfo, an Istio-enabled multi-service application:

  1. Download and configure istioctl


    a. Use Cloud Shell to download and extract the Istio release, with the istioctl tool, and sample code:

      - export LAB_DIR=$HOME/bookinfo-lab
        export ISTIO_VERSION=1.4.6

        mkdir $LAB_DIR
        cd $LAB_DIR

        curl -L https://git.io/getLatestIstio | ISTIO_VERSION=$ISTIO_VERSION sh -


    b. Make the Istio tools visible to your environment, by adding bin to your PATH:

      - cd ./istio-*

        export PATH=$PWD/bin:$PATH

    Result:
    The installation directory contains the following files:

      - Sample applications in samples/.

      - The istioctl client binary in the bin/ directory.



    c. Verify istioctl works:

      - istioctl version



  2. Deploy the Bookinfo application:


    a. Look at the .yaml which describes the bookInfo application:

      - cat samples/bookinfo/platform/kube/bookinfo.yaml

Look for containers to see that each Deployment, has one container, for each version of each service in the Bookinfo application.



    b. Look at the same .yaml with Istio proxy sidecars injected using istioctl:

      - istioctl kube-inject -f
        samples/bookinfo/platform/kube/bookinfo.yaml


Now, when you scroll up, and look for containers, you can see extra configuration describing the proxy sidecar containers that will be deployed.




    c. In Cloud Shell, use the following command to inject the proxy sidecar along with each application Pod that is deployed:

         - kubectl apply -f <(istioctl kube-inject -f   samples/bookinfo/platform/kube/bookinfo.yaml)



  3. Enable external access using an Istio Ingress Gateway;


    a. Look at the .yaml which describes the configuration for the application ingress gateway:

      - cat samples/bookinfo/networking/bookinfo-gateway.yaml



    b. Configure the ingress gateway for the application, which exposes an external IP you will use later:

      - kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml

        Result:

        gateway.networking.istio.io/bookinfo-gateway created
        virtualservice.networking.istio.io/bookinfo created




  4. Verify the Bookinfo deployments:


    a. Confirm that the application has been deployed correctly, review services, pods, and the ingress gateway:

      - kubectl get services



    b. Review running application pods:

      - kubectl get pods


You may need to re-run this command until you see that all six pods are in Running status.



    c. Confirm that the Bookinfo application is running by sending a curl request to it from some pod, within the cluster, for example from ratings:

      - kubectl exec -it $(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}') \
    -c ratings -- curl productpage:9080/productpage | grep -o "<title>.*</title>"

      Result:
        <title>Simple Bookstore App</title>




    d. Confirm the ingress gateway has been created:

      - kubectl get gateway



    e. Get the external IP address of the ingress gateway:

      - kubectl get svc istio-ingressgateway -n istio-system


        Output:
        NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP
        istio-ingressgateway   LoadBalancer   10.107.15.123   104.154.143.236

Your output will provide an External-IP, note it down for use later.



    f. Store the external IP from your output in an environmental variable named GATEWAY_URL:

      - export GATEWAY_URL=[EXTERNAL-IP]



    g. Check that the Bookinfo app is running by sending a curl request to it from outside the cluster:

      - curl -I http://${GATEWAY_URL}/productpage


      Output (Do not Copy):
      HTTP/1.1 200 OK
      content-type: text/html; charset=utf-8
      content-length: 4415
      server: istio-envoy
      ...



Task 3. Use the Bookinfo application:

    1. Point your browser to http://[$GATEWAY_URL]/productpage to see the BookInfo web page. Don't forget to replace [$GATEWAY_URL] with your working external IP address.


    2. Refresh the page several times.
       Notice how you see three different versions of reviews, since we have not yet used Istio to control the version routing.

       There are three different book review services being called in a round-robin style:

       - no stars
       - black stars
       - red stars.
