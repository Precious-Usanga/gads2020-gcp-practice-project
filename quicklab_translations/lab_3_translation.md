# LAB: Google Cloud Fundamentals: Getting Started with GKE

## Objectives:

In this lab, you learn how to perform the following tasks:

    - Provision a Kubernetes cluster using Kubernetes Engine.

    - Deploy and manage Docker containers using kubectl.

## Steps:

1. Task 1: Sign in to the Google Cloud Platform (GCP) Console
For each lab, you get a new GCP project and set of resources for a fixed time at no cost.

    1. Make sure you signed into Qwiklabs using an incognito window.
    2. Note the lab's access time (for example, 02:00:00 and make sure you can finish in that time block.
        There is no pause feature. You can restart if needed, but you have to start at the beginning.
    3. When ready, click Start Lab.
    4. Note your lab credentials. You will use them to sign in to Cloud Platform Console.
    5. Click Open Google Console.
    6. Click Use another account and copy/paste credentials for this lab into the prompts.
        If you use other credentials, you'll get errors or incur charges.
    7. Accept the terms and skip the recovery resource page.
        Do not click End Lab unless you are finished with the lab or want to restart it. This clears your work and removes the project.

2. Task 2: Confirm that needed APIs are enabled

    1. Enter the following to display the project IDs for your Google Cloud projects:

        gcloud projects list
    or

    1. Make a note of the name of your GCP project. This value is shown in the top bar of the Google Cloud Platform Console. It will be of the form qwiklabs-gcp- followed by hexadecimal numbers. store the applicable project ID given to you by Qwiklabs in a variable

            export PROJECT_ID=YOUR_PROJECT_ID

    2. Using the applicable project ID from the previous step, set the default project to the one in which you want to enable the API:

            gcloud config set project PROJECT_ID

    3. Get a list of services that you can enable in your project:

            gcloud services list --available

    4. Scroll down in the list of enabled APIs, and confirm that both of these APIs are enabled:
        - Kubernetes Engine API
        - Container Registry API
        If you don't see the API listed, that means you haven't been granted access to enable the API.

    5. If either API is missing, Using the applicable service name from the previous step, enable the service:

            gcloud services enable Kubernetes Engine API
            gcloud services enable Container Registry API

3. Task 3: Start a Kubernetes Engine cluster

    1. For convenience, place the zone that Qwiklabs assigned you to into an environment variable called MY_ZONE. At the Cloud Shell prompt, type this partial command:

            export MY_ZONE=us-central1-a

    2. Start a Kubernetes cluster managed by Kubernetes Engine. Name the cluster webfrontend and configure it to run 2 nodes:

            gcloud container clusters create webfrontend --zone $MY_ZONE --num-nodes 2
        
        It takes several minutes to create a cluster as Kubernetes Engine provisions virtual machines for you.

    3. After the cluster is created, check your installed version of Kubernetes using the kubectl version command:

            kubectl version
        
        The gcloud container clusters create command automatically authenticated kubectl for you.

4. Task 4: Run and deploy a container

    1. From your Cloud Shell prompt, launch a single instance of the nginx container. (Nginx is a popular web server.)

            kubectl create deploy nginx --image=nginx:1.17.10
    
        In Kubernetes, all containers run in pods. This use of the kubectl create command caused Kubernetes to create a deployment consisting of a single pod containing the nginx container. A Kubernetes deployment keeps a given number of pods up and running even in the event of failures among the nodes on which they run. In this command, you launched the default number of pods, which is 1.
    
        Note: If you see any deprecation warning about future version you can simply ignore it for now and can proceed further.

    2. View the pod running the nginx container:

            kubectl get pods
    
    3. Expose the nginx container to the Internet:

            kubectl expose deployment nginx --port 80 --type LoadBalancer
        
        Kubernetes created a service and an external load balancer with a public IP address attached to it. The IP address remains the same for the life of the service. Any network traffic to that public IP address is routed to pods behind the service: in this case, the nginx pod.
    
    4. View the new service:

            kubectl get services

        You can use the displayed external IP address to test and contact the nginx container remotely.

        It may take a few seconds before the External-IP field is populated for your service. This is normal. Just re-run the kubectl get services command every few seconds until the field is populated.

    5. Open a new web browser tab and paste your cluster's external IP address into the address bar. The default home page of the Nginx browser is displayed.

            curl http://[external_IP]/
    
    6. Scale up the number of pods running on your service:

            kubectl scale deployment nginx --replicas 3
        
        Scaling up a deployment is useful when you want to increase available resources for an application that is becoming more popular.

    7. Confirm that Kubernetes has updated the number of pods:

            kubectl get pods

    8. Confirm that your external IP address has not changed:

            kubectl get services

    9. Return to the web browser tab in which you viewed your cluster's external IP address. Refresh the page to confirm that the nginx web server is still responding.

            curl http://[external_IP]/
    
