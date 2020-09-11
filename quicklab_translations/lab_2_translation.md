# LAB: Google Cloud Fundamentals: Getting Started with Compute Engine

## Objectives:

In this lab, you will learn how to perform the following tasks:

    - Create a Compute Engine virtual machine using the Google Cloud Platform (GCP) Console.

    - Create a Compute Engine virtual machine using the gcloud command-line interface.

    - Connect between the two instances.

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

2. Task 2: Create a virtual machine using the GCP Console

        gcloud compute instances create my-vm-1 --machine-type=n1-standard-1 --zone=us-central1-a --image-project=debian-cloud --image=debian-9-stretch-v20200902  --subnet=default --tags=http-server 

        gcloud compute firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server

3. Task 3: Create a virtual machine using the gcloud command line
    1. List all zones in the assigned region

        gcloud compute zones list | grep us-central1

    2. Choose a zone from that list other than the zone Qwiklabs assigned you.

        gcloud config set compute/zone us-central1-b

    3. Create a VM instance in your choosen zone

        gcloud compute instances create my-vm-2 --machine-type=n1-standard-1 --image-project=debian-cloud --image=debian-9-stretch-v20190213 --subnet=default

4. Task 4: Connect between VM instances
    1.  Connect to my-vm-2 via ssh

            gcloud compute ssh my-vm-2

    2. Use the ping command to confirm that my-vm-2 can reach my-vm-1 over the network
        - ping my-vm-1 from my-vm-2 (abort command after 4th ping):

            ping -c 4 my-vm-1

    3. Use the ssh command to open a command prompt on my-vm-1:

            ssh my-vm-1
        
        If you are prompted about whether you want to continue connecting to a host with unknown authenticity, enter yes to confirm that you do
    
    4. At the command prompt on my-vm-1, install the Nginx web server:

            sudo apt-get install nginx-light -y

    5. Use the nano text editor to add a custom message to the home page of the web server:

            sudo nano /var/www/html/index.nginx-debian.html

    6. Use the arrow keys to move the cursor to the line just below the h1 header. Add text like this, and replace YOUR_NAME with your name:

            Hi from Precious

    7. Press Ctrl+O and then press Enter to save your edited file, and then press Ctrl+X to exit the nano text editor.

    8. Confirm that the web server is serving your new page. At the command prompt on my-vm-1, execute this command:

            curl http://localhost/

        - Result: The response will be the HTML source of the web server's home page, including your line of custom text.

    9. To exit the command prompt on my-vm-1, execute this command:

            exit

        You will return to the command prompt on my-vm-2

    10. To confirm that my-vm-2 can reach the web server on my-vm-1, at the command prompt on my-vm-2, execute this command:

            curl http://my-vm-1/
        
        - Result: The response will again be the HTML source of the web server's home page, including your line of custom text.
    
    11. Copy the External IP address for my-vm-1 and paste it into the address bar of a new browser tab. You will see your web server's home page, including your custom text.

        - Get the External IP address of the VM instance

            gcloud compute instances describe my-vm-1 --format='get(networkInterfaces[0].accessConfigs[0].natIP)'