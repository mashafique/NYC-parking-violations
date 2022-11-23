# Mohammad Shafique
# CIS9760
# Project 1 - Analyzing Millions of NYC Parking Violations

The goal of this project is to apply what we have learned thusfar about EC2, using the terminal, Docker and containerization as well as using Elasticsearch to ingest a large dataset and Kibana to create dashboards via the NYC Parking Violations dataset found [here](https://dev.socrata.com/foundry/data.cityofnewyork.us/nc67-uf89).
 
 The following are required to build and run this project:
 1. An EC2 instance with the appropriate IAM role to allow access to the terminal via the web browser. 
 2. A Elasticsearch domain to upload our dataset to.
 3. Docker to build and run our containers. 
 4. Kibana for dashboarding. 
 5. Python for programming.
 6. An app token aquired [here](https://dev.socrata.com/foundry/data.cityofnewyork.us/nc67-uf89) by making an account.

Steps to build and run this project:
1. Create an IAM role as follows: 
    1. Go to the AWS console and search for IAM.
    2. Click on roles.
    3. Click Create Role.
    4. Select AWS Service and select EC2 from the Choose a Use Case option.
    5. Click next and search for the AmazonEC2RoleforSSM role.
    6. Hit Next: Tags and Next: review. 
    7. Name the role and then click Create role.
2. Create an EC2 instance as follows:
    1. Go to the AWS console and search for EC2.
    2. Click Launch Instance.
    3. Search for and select Ubuntu Server 20.04 (free tier eligble).
    4. Click Next: Configure Instance Details.
    5. For the IAM role option select the IAM role created in the previous step.
    6. Click Next: Add Storage and Next: Add Tags.
    7. Hit Next: Configure Security Group. 
    8. Click Add Rule. 
    9. Enter 5001 for Port Range and select Anywhere for the Source.
    10. Click Launch, Proceed without a Key Pair and Launch Instances.
3. Launch terminal in the web browser as follows:
    1. Go the EC2 page and click on your instance. 
    2. Copy the Public DNS.
    3. Search for Systems Manager in the AWS Console.
    4. Click on Sessions Manager on the lefthand menu.
    5. Click on your instance and then press Start Session. A terminal will open up.
    6. Type `sudo su` and hit enter.
    7. Type the following to install an in-browser file editor:
        ```
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
        sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
        sudo apt-get update
        apt-cache policy docker-ce
        sudo apt-get install -y docker-ce
        sudo systemctl status docker
        sudo groupadd docker
        sudo gpasswd -a $USER docker
        newgrp docker
        ```
    8. Type `docker run -dv "$PWD":/srv -p 5001:80 filebrowser/filebrowser` and hit enter.
    9. In a new tab paste the Public DNS and add :5001/ after it as follows: `http://[URL]:5001/` An example URL would be:
        `http://ec2-3-14-11-96.us-east-2.compute.amazonaws.com:5001/`
    10. This will take you to a login page. The default username is admin and the default password is admin. 
    11. On this page on the top-right corner there is an upwards pointing arrow. Click this arrow and upload the project folder. 
4. Create an Elasticsearch domain as follows:
    1. Search for Elasticsearch on the AWS Console and select Amazon OpenSearch Service. 
    2. Click on Create Domain. 
    3. Give a name to your domain. 
    4. Select Develop and testing.
    5. Select the lastest version.
    6. Select 3 for the number of nodes.
    7. Select Public access under the Network tab.
    8. Select the checkbox for Enable fine-grained access control and the create master user radio button under the Fine-grained access control tab. Create a master username and password. This will be needed later to run our docker container.
    9. Under the Access Policy tab, select Only use fine-grained access control.
    10. Click Create. 
    11. Once the domain is created there will be a Domain endpoint available. Take note of this as it will be needed to run the Docker container. Also take note of the of the OpenSearch Dashboards URL as this is how we will access Kibana. 
5. Type `cd project01` in the terminal to enter the project folder. 
6. Type `docker build -t project1:1.0 .` to build our container. 
7. Type the following:
    `docker run -e DATASET_ID="nc67-uf89" -e APP_TOKEN=<app_token> -e ES_HOST=<domain_endpoint> -e ES_USERNAME=<master_username> -e ES_PASSWORD=<master_password> -e INDEX_NAME="parking" bigdataproject1:1.0 --page_size=1200 --num_pages=1000`
    - APP_TOKEN is the token we created in an earlier step. 
    - DATASET_ID is the ID of our dataset. 
    - bigdataproject1:1.0 is the name of our Docker image.
    - ES_HOST is the Elasticsearch Domain 
    - ES_USERNAME is the master username we created when creating our Elasticsearch domain.
    - ES_PASSWORD is the master password we created when creating our Elasticsearch domain.
    - --page_size is the number of records that we want to request. This argument is required.
    - --num_pages is the number of times we want to query for data. This arguement is optional. 
    - INDEX_NAME is the name of our Elasticsearch index. 
8. To see how many records were loaded go to `ES_HOST/<INDEX_NAME>/_count`.
9. Dashboards can be created via Kibana as follows:
    1. Enter the OpenSearch Dashboards URL into the browser and log in using the master username and password created when we were creating our Elasticsearch domain.
    2. Click Explore on my own.
    3. Click Global and click Confirm.
    4. Click Discover on the lefthand menu and click Create index pattern.
    5. Search for the INDEX_NAME that you entered when running the container. 
    6. Select a timefield. For this dataset the issue_date is a good option.
    7. On the top-right menu change the data range to 20 years ago and hit refresh to see our data.
    8. On the lefthand menu you can now hit Dashboard or Visualize to create dashboards and visualizations. 
