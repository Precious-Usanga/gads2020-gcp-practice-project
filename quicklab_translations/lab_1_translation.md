# LAB: Google Cloud Fundamentals: Getting Started with Cloud Storage and Cloud SQL

## Objectives:
In this lab, you learn how to perform the following tasks:

    -    Create a Cloud Storage bucket and place an image into it.

    -    Create a Cloud SQL instance and configure it.

    -    Connect to the Cloud SQL instance from a web server.

    -    Use the image in the Cloud Storage bucket on a web page.

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

2. Task 2: Deploy a web server VM instance

    - create VM instance that allows http traffic and runs a startup script that installs apache,php, php-mysql

            gcloud compute instances create bloghost --zone=us-central1-a --machine-type=n1-standard-1 --image-project=debian-cloud --image=debian-9-stretch-v20200902 --subnet=default --tags=http-server --metadata=startup-script=apt-get\ update$'\n'apt-get\ install\ apache2\ php\ php-mysql\ -y$'\n'service\ apache2\ restart --maintenance-policy=MIGRATE   

    - configure firewall rules to allow http traffic
    
            gcloud compute firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server

        Instance can take about two minutes to launch and be fully available for use.
    
    - Copy the bloghost VM instance's internal and external IP addresses to a text editor for use later in the lab.

        - list the instances using:

            gcloud compute instances list --zones=us-central1-a

        - Copy and store the internal and external IP addresses of bloghost in the environment variable INTERNAL_IP and EXTERNAL_IP respectively as follows:

            export INTERNAL_IP=COPIED_INTERNAL_IP
            export EXTERNAL_IP=COPIED_EXTERNAL_IP

3. Task 3: Create a Cloud Storage bucket using the gsutil command line

    All Cloud Storage bucket names must be globally unique. To ensure that your bucket name is unique, these instructions will guide you to give your bucket the same name as your Cloud Platform project ID, which is also globally unique.

    Cloud Storage buckets can be associated with either a region or a multi-region location: US, EU, or ASIA. In this activity, you associate your bucket with the multi-region closest to the region and zone that Qwiklabs or your instructor assigned you to.


    1. For convenience, enter your chosen location into an environment variable called LOCATION. Enter one of these commands:

            export LOCATION=US
            Or
            export LOCATION=EU
            Or
            export LOCATION=ASIA

    2. In Cloud Shell, the DEVSHELL_PROJECT_ID environment variable contains your project ID. Enter this command to make a bucket named after your project ID:

            gsutil mb -l $LOCATION gs://$DEVSHELL_PROJECT_ID
    
    3. Retrieve a banner image from a publicly accessible Cloud Storage location:

            gsutil cp gs://cloud-training/gcpfci/my-excellent-blog.png my-excellent-blog.png

    4. Copy the banner image to your newly created Cloud Storage bucket:

            gsutil cp my-excellent-blog.png gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png

    5. Modify the Access Control List of the object you just created so that it is readable by everyone:

            gsutil acl ch -u allUsers:R gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png

4. Task 4: Create the Cloud SQL instance

    1. Create a MySql Database Engine with instance ID 'blog-db', Set the region and zone as assigned to you by Qwiklabs. The best performance is achieved by placing the client and the database close to each other.

            gcloud sql instances create blog-db --region=us-central1 --zone=us-central1-a --user=root

    2. Set the password for the "root@%" MySql user with this command. enter your password into the [PASSWORD] placeholder. Set a password you'll remember.

            gcloud sql users set-password root --host=% --instance blog-db --password [PASSWORD]
    
    Wait for the instance to finish deploying. It will take a few minutes.
    
    3. From the SQL instances details, copy the Public IP address for your SQL instance to a text editor for use later in this lab.

            gcloud sql instances describe blog-db

    4. Add User account with username=blogdbuser and password of your choice in the [PASSWORD] placeholder

            gcloud sql users create blogdbuser --host=% --instance=blog-db --password=[PASSWORD]
        
        Wait for the user to be created.

    5. Add network to SQL Instance
        - Add an IPv4 address to the instance:

            gcloud sql instances patch blog-db --assign-ip

        - Show all existing authorized addresses by describing the instance:

            gcloud sql instances describe blog-db

        Look for authorizedNetwork entries under ipConfiguration, and note the authorized address you want to keep.

        - Authorize the network with the external IP address of your bloghost VM instance, followed by /32: (remember we stored our VM instance external IP in an env variable EXTERNAL_IP)

            gcloud sql instances patch blog-db --authorized-networks=$EXTERNAL_IP/32
        
        Be sure to use the external IP address of your VM instance followed by /32. Do not use the bloghost VM instance's internal IP address.

        - Confirm your changes

            gcloud sql instances describe blog-db

5. Task 5: Configure an application in a Compute Engine instance to use Cloud SQL

    1. SSH in the row for your VM instance bloghost

            gcloud compute ssh bloghost

    2. In your ssh session on bloghost, change your working directory to the document root of the web server:

            cd /var/www/html

    3. Use the nano text editor to edit a file called index.php:

            sudo nano index.php
    
    4. Paste the content below into the file:

            <html>
                <head><title>Welcome to my excellent blog</title></head>
                <body>
                <h1>Welcome to my excellent blog</h1>
                <?php
                $dbserver = "CLOUDSQLIP";
                $dbuser = "blogdbuser";
                $dbpassword = "DBPASSWORD";
                // In a production blog, we would not store the MySQL
                // password in the document root. Instead, we would store it in a
                // configuration file elsewhere on the web server VM instance.

                $conn = new mysqli($dbserver, $dbuser, $dbpassword);

                if (mysqli_connect_error()) {
                        echo ("Database connection failed: " . mysqli_connect_error());
                } else {
                        echo ("Database connection succeeded.");
                }
                ?>
                </body>
            </html>

        In a later step, you will insert your Cloud SQL instance's IP address and your database password into this file. For now, leave the file unmodified.

    5. Press Ctrl+O, and then press Enter to save your edited file.

    6. Press Ctrl+X to exit the nano text editor.

    7. Restart the web server:

            sudo service apache2 restart

    8. Open a new web browser tab and paste into the address bar your bloghost VM instance's external IP address followed by /index.php. The URL will look like this: 
    
            35.192.208.2/index.php

    or on the command line, enter this command (remember we stored our VM instance external IP in an env variable EXTERNAL_IP):

            curl http://$EXTERNAL_IP/index.php

        Be sure to use the external IP address of your VM instance followed by /index.php. Do not use the VM instance's internal IP address. Do not use the sample IP address shown here.

    When you load the page, you will see that its content includes an error message beginning with the words:

            Database connection failed: ...

        This message occurs because you have not yet configured PHP's connection to your Cloud SQL instance.

    9. Return to your ssh session on bloghost. Use the nano text editor to edit index.php again

            sudo nano index.php

    10. In the nano text editor, replace CLOUDSQLIP with the Cloud SQL instance Public IP address that you noted above. Leave the quotation marks around the value in place.

    11. In the nano text editor, replace DBPASSWORD with the Cloud SQL database password that you defined above. Leave the quotation marks around the value in place.

    12. Press Ctrl+O, and then press Enter to save your edited file.

    13. Press Ctrl+X to exit the nano text editor.

    14. Restart the web server:

            sudo service apache2 restart

    15. Return to the web browser tab in which you opened your bloghost VM instance's external IP address. When you load the page, the following message appears:

            Database connection succeeded.

        In an actual blog, the database connection status would not be visible to blog visitors. Instead, the database connection would be managed solely by the administrator.

6. Task 6: Configure an application in a Compute Engine instance to use a Cloud Storage object

    1. list buckets in your gcp project

            gsutil ls

    2. Copy the name of the bucket that is named after your GCP project and store in an environment variable

            export BUCKET_NAME=YOUR_COPIED_BUCKET_NAME

    3. list all objects in the bucket

            gsutil ls -r gs://$BUCKET_NAME/**
        
        - result:

            gs://qwiklabs-gcp-0005e186fa559a09/my-excellent-blog.png 
    
    4. To view the metadata associated with the object:

            gsutil stat gs://$BUCKET_NAME/my-excellent-blog.png 

        - result if successful:

            gs://my-awesome-bucket/cat.jpeg:
                Creation time:          Fri, 03 Feb 2017 22:43:31 GMT
                Update time:            Wed, 10 May 2017 18:33:07 GMT
                Storage class:          STANDARD
                Content-Length:         11012
                Content-Type:           image/jpeg
                Metadata:
                    Breed:              Tabby
                Hash (crc32c):          HQbzrB==
                Hash (md5):             OBydg25+pPG1Cwawjsl7DA==
                ETag:                   CJCh9apA9dECAEs=
                Generation:             1486161811706000
                Metageneration:         11

    5. Create and Copy a URI to access your object in the bucket via API links like this:

            - format: https://storage.googleapis.com/$BUCKET_NAME/OBJECT_NAME

            - result: https://storage.googleapis.com/qwiklabs-gcp-0005e186fa559a09/my-excellent-blog.png

            to Check, run this command:

                curl https://storage.googleapis.com/qwiklabs-gcp-0005e186fa559a09/my-excellent-blog.png

            If you dont see the above, confirm that your attempt to change the object's Access Control list with the gsutil acl ch command was successful.

                gsutil acl ch -u allUsers:R gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png

                In Cloud Shell, the DEVSHELL_PROJECT_ID environment variable contains your project ID

    6. Return to your ssh session on your bloghost VM instance.

    7. Enter this command to set your working directory to the document root of the web server:

            cd /var/www/html

    8. Use the nano text editor to edit index.php:

            sudo nano index.php

    9. Use the arrow keys to move the cursor to the line that contains the h1 element. Press Enter to open up a new, blank screen line, and then paste the URL you copied earlier into the line.

    10. Paste this HTML markup immediately before the URL:

            <img src='

    11. Place a closing single quotation mark and a closing angle bracket at the end of the URL:

            '>
        
        The resulting line will look like this:

            <img src='https://storage.googleapis.com/qwiklabs-gcp-0005e186fa559a09/my-excellent-blog.png'>

        The effect of these steps is to place the line containing <img src='...'> immediately before the line containing <h1>...</h1>

        Do not copy the URL shown here. Instead, copy the URL shown on the command line after you ran the command: gsutil stat gs://$BUCKET_NAME/my-excellent-blog.png 

    12. Press Ctrl+O, and then press Enter to save your edited file

    13. Press Ctrl+X to exit the nano text editor.

    14. Restart the web server:

            sudo service apache2 restart
    
    15. Return to the web browser tab in which you opened your bloghost VM instance's external IP address. When you load the page, its content now includes a banner image.
