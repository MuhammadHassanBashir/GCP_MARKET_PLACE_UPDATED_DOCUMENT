This guide provides instructions on how to set up a complete project in GCP.

## Step 1: Authenticate with Google Cloud:
Use the following command to authenticate your local environment with your Google Cloud account:

    gcloud auth login

This command will open a web browser where you can log in with your Google account.

After authentication, set your active Google Cloud project by running:

    gcloud config set project YOUR_GCP_PROJECT_ID

Replace YOUR_GCP_PROJECT_ID with your actual Google Cloud project ID.  

## Creating and Configuring the Service Account using gcloud commands.

## Step 2: Create the Service Account

Use the following gcloud command to create a new service account:

    gcloud iam service-accounts create terraform \
    --display-name="terraform"
    
## Step 3: Assign IAM Role Permissions

Assign the required IAM roles to the service account with the following command:     
    
    gcloud projects add-iam-policy-binding "YOUR_GCP_PROJECT_ID" \
      --member="serviceAccount:terraform@YOUR_GCP_PROJECT_ID.iam.gserviceaccount.com" \
      --role="roles/compute.admin"

  gcloud projects add-iam-policy-binding "YOUR_GCP_PROJECT_ID" \
    --member="serviceAccount:terraform@YOUR_GCP_PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/compute.storageAdmin"

  gcloud projects add-iam-policy-binding "YOUR_GCP_PROJECT_ID" \
    --member="serviceAccount:terraform@YOUR_GCP_PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/editor"

  gcloud projects add-iam-policy-binding "YOUR_GCP_PROJECT_ID" \
    --member="serviceAccount:terraform@YOUR_GCP_PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/container.admin"

  gcloud projects add-iam-policy-binding "YOUR_GCP_PROJECT_ID" \
    --member="serviceAccount:terraform@YOUR_GCP_PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/monitoring.viewer"

  gcloud projects add-iam-policy-binding "YOUR_GCP_PROJECT_ID" \
    --member="serviceAccount:terraform@YOUR_GCP_PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/secretmanager.admin"

  gcloud projects add-iam-policy-binding "YOUR_GCP_PROJECT_ID" \
    --member="serviceAccount:terraform@YOUR_GCP_PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/secretmanager.secretAccessor"

  gcloud projects add-iam-policy-binding "YOUR_GCP_PROJECT_ID" \
    --member="serviceAccount:terraform@YOUR_GCP_PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/iam.securityAdmin"

  gcloud projects add-iam-policy-binding "YOUR_GCP_PROJECT_ID" \
    --member="serviceAccount:terraform@YOUR_GCP_PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/vpcaccess.admin"

  gcloud projects add-iam-policy-binding "YOUR_GCP_PROJECT_ID" \
    --member="serviceAccount:terraform@YOUR_GCP_PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/iam.serviceAccountAdmin"

  gcloud projects add-iam-policy-binding "YOUR_GCP_PROJECT_ID" \
    --member="serviceAccount:terraform@YOUR_GCP_PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/storage.admin"

  gcloud projects add-iam-policy-binding "YOUR_GCP_PROJECT_ID" \
    --member="serviceAccount:terraform@YOUR_GCP_PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/iam.workloadIdentityUser"

  gcloud projects add-iam-policy-binding "YOUR_GCP_PROJECT_ID" \
    --member="serviceAccount:terraform@"YOUR_GCP_PROJECT_ID".iam.gserviceaccount.com" \
    --role="roles/servicenetworking.networksAdmin"


## Step 4: Generate the Service Account Key
  
Generate the service account key and save it to a JSON file:
    
    gcloud iam service-accounts keys create ./secret.json \
    --iam-account=terraform@"YOUR_GCP_PROJECT_ID".iam.gserviceaccount.com

After generating the new key, activate the service account using:

    gcloud auth activate-service-account --key-file=secret.json

## Step 5: Create a Bucket for Storing the Terraform State File

    gcloud storage buckets create gs://NAME-OF-YOUR-GCP-BUCKET --location=us-central1

Remember, the bucket name must be globally unique. After creating the bucket, replace the bucket name in the terraform backend section within the main.tf file.

## Step 6: Creating Infrastructure in GCP using Terraform

    terraform init
    terraform plan -var="projectName=YOUR_GCP_PROJECT_ID"
    terraform apply -var="projectName=YOUR_GCP_PROJECT_ID" -auto-approve
 
## Step 7: Workload Identity setup

Connecting the cluster 

    gcloud container clusters get-credentials disearch-cluster --zone us-central1-c --project YOUR_GCP_PROJECT_ID

    kubectl create serviceaccount gke-sa --namespace=default

    gcloud iam service-accounts add-iam-policy-binding gke-sa@YOUR_GCP_PROJECT_ID.iam.gserviceaccount.com \
    --role roles/iam.workloadIdentityUser \
    --member "serviceAccount:$YOUR_GCP_PROJECT_ID.svc.id.goog[default/gke-sa]"
    
    kubectl annotate serviceaccount gke-sa \
    --namespace default \
    iam.gke.io/gcp-service-account=gke-sa@YOUR_GCP_PROJECT_ID.iam.gserviceaccount.com
    
## Step 8: Fetch and saving the private IP address of the Cloud SQL instance to GCP secrets

    gcloud secrets versions add DB_HOST --data-file=<(gcloud sql instances describe disearch-db --format="json(ipAddresses)" | jq -r '.ipAddresses[] | select(.type == "PRIVATE") | .ipAddress')

## Step 9: Creating Postgres Connection String 

    ENCODED_CONN_STRING=$(echo -n "postgresql://postgres:$(gcloud secrets versions access latest --secret=DB_PASSWORD)@$(gcloud secrets versions access latest --secret=DB_HOST)/postgres" | base64 -w 0)

Use below commands for Verificaiton

    echo "ENCODED_CONN_STRING: $(echo "$ENCODED_CONN_STRING" | base64 --decode)"

## Step 10: Cloud Function Deployments

    BUCKET_NAME=$(gcloud secrets versions access latest --secret="GCP_BUCKET") && echo "Bucket name: $BUCKET_NAME"

Clone the GitHub repository. 

Email at aareez.asif@aretecinc.com to get the authentication token for pulling the cloud functions from the repository.

    git clone https://PASTE_AUTHENTICATION_TOKEN_HERE@github.com/Aretec-Inc/uploader-trigger-cf.git
    git clone https://PASTE_AUTHENTICATION_TOKEN_HERE@github.com/Aretec-Inc/document-status-cf.git
    git clone https://PASTE_AUTHENTICATION_TOKEN_HERE@github.com/Aretec-Inc/image-process-cf.git
    git clone https://PASTE_AUTHENTICATION_TOKEN_HERE@github.com/Aretec-Inc/metadata-extractor-cf.git
    git clone https://PASTE_AUTHENTICATION_TOKEN_HERE@github.com/Aretec-Inc/pdf-convert-cf.git

Email at aareez.asif@aretecinc.com to get the authentication token for pulling the cloud functions from the repository.

Appling Policy
    
    GCS_SERVICE_ACCOUNT=$(gsutil kms serviceaccount -p "YOUR_GCP_PROJECT_NUMBER") && echo "GCS_SERVICE_ACCOUNT: $GCS_SERVICE_ACCOUNT"

    gcloud projects add-iam-policy-binding "YOUR_GCP_PROJECT_ID" \
      --member serviceAccount:$GCS_SERVICE_ACCOUNT \
      --role roles/pubsub.publisher

Cloud Functions

Deploying Cloud Function uploader-trigger

    echo "Deploying uploader-trigger"
    cd uploader-trigger-cf
    git checkout deployment
    gcloud functions deploy uploader-trigger \
      --runtime python39 \
      --entry-point main_func \
      --set-env-vars=LOG_EXECUTION_ID=true \
      --set-secrets=bucket=projects/$PROJECT_ID/secrets/GCP_BUCKET:latest,project_id=projects/"YOUR_GCP_PROJECT_ID"/secrets/project_id:latest \
      --service-account gke-sa@"YOUR_GCP_PROJECT_ID".iam.gserviceaccount.com \
      --serve-all-traffic-latest-revision \
      --vpc-connector disearch-vpc-connector \
      --trigger-bucket $BUCKET_NAME \
      --region us-central1 \
      --gen2 \
      --quiet
    cd ..
    
Deploying Cloud Function document-status

    echo "Deploying document-status"
    cd document-status-cf
    git checkout deployment
    gcloud functions deploy document-status \
      --runtime python39 \
      --trigger-http \
      --entry-point main_func \
      --set-secrets 'DB_HOST=DB_HOST:latest,DB_USER=DB_USER:latest,DB_PASSWORD=DB_PASSWORD:latest' \
      --service-account  gke-sa@"YOUR_GCP_PROJECT_ID".iam.gserviceaccount.com \
      --serve-all-traffic-latest-revision \
      --vpc-connector disearch-vpc-connector \
      --region us-central1 \
      --gen2 \
      --quiet
    cd ..

Deploying Cloud Function image-processing

    echo "Deploying image-processing"
    cd image-process-cf
    git checkout deployment
    gcloud functions deploy image-processing \
      --runtime python39 \
      --entry-point main_func \
      --service-account  gke-sa@"YOUR_GCP_PROJECT_ID".iam.gserviceaccount.com \
      --set-env-vars=SCHEMA=disearch_search,LOG_EXECUTION_ID=true \
      --set-secrets 'LOG_INDEX=LOG_INDEX:latest,LOGS_URL=LOGS_URL:latest,gpt_key=OPENAI_API_KEY:latest' \                  ## remember to give the open api key in terraform variable.tf
      --vpc-connector disearch-vpc-connector \
      --serve-all-traffic-latest-revision \
      --trigger-http \
      --gen2 \
      --region us-central1 \
      --memory 2G
    cd ..  

Deploying Cloud Function Metadata Extractor
    
    echo "Deploying Metadata Extractor"
    cd metadata-extractor-cf
    git checkout deployment
    gcloud functions deploy update_metadata_ingested_document \
      --runtime python39 \
      --trigger-http \
      --entry-point main_func \
      --set-env-vars=LOG_EXECUTION_ID=true \
      --set-secrets=project_id=projects/$PROJECT_ID/secrets/project_id:latest \
      --service-account gke-sa@"YOUR_GCP_PROJECT_ID".iam.gserviceaccount.com \
      --vpc-connector disearch-vpc-connector --region us-central1 --serve-all-traffic-latest-revision --gen2 --memory 1G
    cd ..

## Step 11: RUN the deployer image from GCP Market place for deploying the Applications services.

  When deploying the deployer image from Google Cloud Marketplace, you need to provide following variables.

  - GCP project id
  - GCP Cloud SQL Db Connection String in Base64 Encoded format
  - GKE kube API Server URL in Base64 Encoded format
  - Website URL
  - Client Email Address
  - DB User
  - DB Password 
  - DB Host
  - GCP Bucket Name created by terraform 
  - Cloudfunction URLs
    Name: 
      document-status 
      image-processing 
      update_metadata_ingested_document  
  
  You can obtain the required variable information using the following commands.

  - GCP project id: Your GCP Project ID
  - GCP Cloud SQL Db Connection String in Base64 Encoded format

    Creating Postgres Connection String

      ENCODED_CONN_STRING=$(echo -n "postgresql://postgres:$(gcloud secrets versions access latest --secret=DB_PASSWORD)@$(gcloud secrets versions access latest --secret=DB_HOST)/postgres" | base64 -w 0)

    Use below commands for Verificaiton

      echo "$ENCODED_CONN_STRING"
  

      echo "ENCODED_CONN_STRING: $(echo "$ENCODED_CONN_STRING" | base64 --decode)"

  - GKE kube API Server URL in Base64 Encoded format

    Fetch private endpoint of Cluster

      echo "INTERNAL_ENDPOINT: $(echo -n "https://$(gcloud container clusters describe disearch-cluster --zone us-central1-c --format="get(privateClusterConfig.privateEndpoint)")" | base64)"

    Decode the base64 encoded endpoint for verfication

      echo <INTERNAL_ENDPOINT_ENCODED_VALUE> | base64 --decode
  
  - Website URL: CLIENT WEBSITE URL
  - Client Email Address: EMAIL ADDRESS
  
  - DB User

    gcloud secrets versions access latest --secret="DB_USER"
 
  - DB Password 

    gcloud secrets versions access latest --secret="DB_PASSWORD"

  - DB Host

    gcloud secrets versions access latest --secret="DB_HOST"

  - GCP Bucket 

    gcloud secrets versions access latest --secret="GCP_BUCKET"

  - Cloudfunction URLs
    Name: 
      document-status:

        gcloud functions describe document-status --region=us-central1 --format="value(url)"

      image-processing:

        gcloud functions describe image-processing --region=us-central1 --format="value(url)"

      update_metadata_ingested_document:

        gcloud functions describe update_metadata_ingested_document --region=us-central1 --format="value(url)"



## Step 12: Updating Cloudfunction URL's in GCP Secrets

    gcloud secrets versions add status_cloud_fn --data-file=<(echo -n "$(gcloud functions describe document-status --region=us-central1 --format="value(url)")") --project="YOUR_GCP_PROJECT_ID"

    gcloud secrets versions add image_summary_cloud_fn --data-file=<(echo -n "$(gcloud functions describe image-processing --region=us-central1 --format="value(url)")") --project="YOUR_GCP_PROJECT_ID"
        
    gcloud secrets versions add metadata_cloud_fn --data-file=<(echo -n "$(gcloud functions describe update_metadata_ingested_document --region=us-central1 --format="value(url)")") --project="YOUR_GCP_PROJECT_ID"


## Step 13: Creating Pubsub and subscriptions

Creating and listing pubsub topic

    gcloud pubsub topics create file-process-topic
    gcloud pubsub topics create image-topic
    gcloud pubsub topics create metadata-topic
    gcloud pubsub topics create pdf-convert-topic
    gcloud pubsub topics create dead-letter-topic
    gcloud pubsub topics list

Creating Pubsub Subscriptions

    gcloud pubsub subscriptions create file-process-subscription \
        --topic=file-process-topic \
        --message-retention-duration=7d \
        --expiration-period=31d \
        --ack-deadline=600
    
    
    gcloud pubsub subscriptions create image-subscription \
        --topic=image-topic \
        --message-retention-duration=7d \
        --expiration-period=31d \
        --ack-deadline=600
    
    
    gcloud pubsub subscriptions create metadata-subscription \
        --topic=metadata-topic \
        --message-retention-duration=7d \
        --expiration-period=31d \
        --ack-deadline=600
    
    
    gcloud pubsub subscriptions create pdf-subscription \
        --topic=pdf-convert-topic \
        --message-retention-duration=7d \
        --expiration-period=31d \
        --ack-deadline=600
## Step 14: Fetching and updating GKE variables in GCP secrets

Connecting the cluster

    gcloud container clusters get-credentials disearch-cluster --zone us-central1-c --project "YOUR_GCP_PROJECT_ID"

    kubectl get service doc-chat-service -o=jsonpath='http://{.status.loadBalancer.ingress[0].ip}' | gcloud secrets versions add DOCUMENT_CHAT_URL --data-file=-
    
    kubectl get service vertex-ai-followup-service -o=jsonpath='http://{.status.loadBalancer.ingress[0].ip}' | gcloud secrets versions add vertexai_followup --data-file=-
    
    kubectl get service vertexai-citation-service -o=jsonpath='http://{.status.loadBalancer.ingress[0].ip}' | gcloud secrets versions add vertexai_citation --data-file=-
    
    kubectl get service vertexai-service -o=jsonpath='http://{.status.loadBalancer.ingress[0].ip}' | gcloud secrets versions add vertexai_python_url --data-file=-
    
    kubectl get service vertexai-summary-service -o=jsonpath='http://{.status.loadBalancer.ingress[0].ip}' | gcloud secrets versions add vertexai-summary --data-file=-
    
    kubectl get service pdflb -o=jsonpath='http://{.status.loadBalancer.ingress[0].ip}' | gcloud secrets versions add pdf_loadbalancer --data-file=-

## Step 15: Updating GCP Secret for Website URL

       echo -n "YOUR_WEBSITE_URL" | gcloud secrets versions add vertexai-referer --data-file=- --project=YOUR_GCP_PROJECT_ID

Make sure to add the Website URL here.

## Step 16: Replacing Values For Cors

    sed -i "s|REPLACE_WITH_WEBSITE_URL|"YOUR_WEBSITE_URL"|g" cors.json
    gsutil cors set cors.json gs://$BUCKET_NAME
    gsutil cors get gs://$BUCKET_NAME

## Step 17: applying Cloud Run

    gcloud run deploy file-upload-pubsub --image=gcr.io/aretecinc-public/disearch/file-upload-pubsub:1.0.8 --set-env-vars=batch_size=1,project_id="YOUR_GCP_PROJECT_ID"  --platform managed  --vpc-connector=projects/"YOUR_GCP_PROJECT_ID"/locations/us-central1/connectors/disearch-vpc-connector --set-secrets=vertexai_python_url=vertexai_python_url:latest,metadata_cloud_fn=metadata_cloud_fn:latest,image_summary_cloud_fn=image_summary_cloud_fn:latest,APP_VERSION=APP_VERSION:latest --region=us-central1 --project="YOUR_GCP_PROJECT_ID" --service-account=gke-sa@"YOUR_GCP_PROJECT_ID".iam.gserviceaccount.com --allow-unauthenticated --min-instances=1 --max-instances=2
    
    
    gcloud run deploy image-pubsub --image=gcr.io/aretecinc-public/disearch/image-pubsub:1.0.8 --set-env-vars=image_subscription_id=image-subscription,project_id="YOUR_GCP_PROJECT_ID",batch_size=1,project_id="YOUR_GCP_PROJECT_ID"  --platform managed  --vpc-connector=projects/"YOUR_GCP_PROJECT_ID"/locations/us-central1/connectors/disearch-vpc-connector --set-secrets=vertexai_python_url=vertexai_python_url:latest,metadata_cloud_fn=metadata_cloud_fn:latest,image_summary_cloud_fn=image_summary_cloud_fn:latest,APP_VERSION=APP_VERSION:latest --region=us-central1 --project="YOUR_GCP_PROJECT_ID" --service-account=gke-sa@"YOUR_GCP_PROJECT_ID".iam.gserviceaccount.com --allow-unauthenticated --min-instances=1 --max-instances=2
    
  
    gcloud run deploy metadata-pubsub --image=gcr.io/aretecinc-public/disearch/metadata-pubsub:1.0.8 --set-env-vars=project_id="YOUR_GCP_PROJECT_ID",batch_size=1 --platform managed  --vpc-connector=projects/"YOUR_GCP_PROJECT_ID"/locations/us-central1/connectors/disearch-vpc-connector --set-secrets=vertexai_python_url=vertexai_python_url:latest,metadata_cloud_fn=metadata_cloud_fn:latest,image_summary_cloud_fn=image_summary_cloud_fn:latest,APP_VERSION=APP_VERSION:latest --region=us-central1 --project="YOUR_GCP_PROJECT_ID" --service-account=gke-sa@"YOUR_GCP_PROJECT_ID".iam.gserviceaccount.com --allow-unauthenticated --min-instances=1 --max-instances=2
    
   
    gcloud run deploy pdf-convert-pubsub --image=gcr.io/aretecinc-public/disearch/pdf-convert-pubsub:1.0.8 --set-env-vars=batch_size=1,project_id="YOUR_GCP_PROJECT_ID" --platform managed  --vpc-connector=projects/"YOUR_GCP_PROJECT_ID"/locations/us-central1/connectors/disearch-vpc-connector --set-secrets=pdf_loadbalancer=pdf_loadbalancer:latest,APP_VERSION=APP_VERSION:latest --region=us-central1 --project="YOUR_GCP_PROJECT_ID" --service-account=gke-sa@"YOUR_GCP_PROJECT_ID".iam.gserviceaccount.com --allow-unauthenticated --min-instances=1 --max-instances=2
    
    
    gcloud run deploy disearch \
    --image=gcr.io/aretecinc-public/disearch/disearch:1.0.8 \
    --set-env-vars=ALLOWED_ORIGIN="YOUR_WEBSITE_URL" --set-env-vars=appName=disearch --set-env-vars=schema=disearch --set-env-vars=AUTH_EMAIL="YOUR_DEFAULT_OWNER_EMAIL" --set-env-vars=IS_PENSDOWN=false --set-env-vars=IS_CONTEXT=false \
    --set-cloudsql-instances="YOUR_GCP_PROJECT_ID":us-central1:disearch-db \
    --service-account=gke-sa@"YOUR_GCP_PROJECT_ID".iam.gserviceaccount.com \
    --set-secrets=service_key=service_key:latest,DB_USER=DB_USER:latest,GCP_BUCKET=GCP_BUCKET:latest,DB_HOST=DB_HOST:latest,DB_PASSWORD=DB_PASSWORD:latest,DOCUMENT_CHAT_URL=DOCUMENT_CHAT_URL:latest,LOG_INDEX=LOG_INDEX:latest,project_id=project_id:latest,vertex_ai_search_summary_url=vertexai-summary:latest,vertex_ai_followup_url=vertexai_followup:latest,IS_VERTEX_AI=IS_VERTEXAI:latest,vertex_ai_search_base_url=vertexai_citation:latest,vertex_ai_init_base_url=vertexai_python_url:latest,APP_VERSION=APP_VERSION:latest \
    --no-cpu-boost \
    --region=us-central1 \
    --project="YOUR_GCP_PROJECT_ID"

## Step 18: Domain Mapping
    
    gcloud beta run domain-mappings create --service disearch --domain "YOUR_WEBSITE_URL" --region us-central1

Remember to add this CNAME record in cloud DNS
    
    ghs.googlehosted.com.

## Step 19: LLM 

    echo "Adding LLM Models on Cloud SQL Instance"
    gcloud run jobs create llm-model-migration --image=$LLM_MODEL_IMPORT_ID \
      --vpc-connector=disearch-vpc-connector --set-cloudsql-instances=$PROJECT_ID:us-central1:disearch-db \
      --region=us-central1 --set-secrets=DB_HOST=DB_HOST:latest,DB_PASSWORD=DB_PASSWORD:latest \
      --project=$PROJECT_ID --service-account=terraform@$PROJECT_ID.iam.gserviceaccount.com

    gcloud run jobs execute llm-model-migration --region us-central1

## What we have done in deployer image


    According to the deployer image documentation, we have created a directory containing 2 files and 1 folder.
    
    Folder Name: chart
    In this folder, we have the main chart named disearch. The templates for the disearch chart are placed in the disearch/chart/templates/ folder, and the values are set in the disearch/chart/values.yaml file. Additionally, inside the main disearch chart folder, we have other sub-charts like etcd, gke-templates, keda, and redis, which are stored in the disearch/chart/charts/ folder.
    
    schema.yaml File:
    This file defines the **release version, images, and variable information** for the Helm chart’s values.yaml file. According to the deployer image documentation, the schema.yaml file maps to the Helm chart’s values.yaml file, allowing Helm to take the values from values.yaml. If a value is specified in both the values.yaml file and the schema.yaml file, the value from the schema.yaml file will override the corresponding value in values.yaml. The schema.yaml essentially provides the values to the Helm chart through the values.yaml file. In schema file we can provide the variable in 2 way. In 1 we are asking the USER for giving the information, basicaly GCP which show to the user in market place UI terminal and user will give the information according to needs and in other way we are giving information by own on self. All thing we will do in **schema.yaml file under properites section**.
    
    Dockerfile:
    In this file, we are copying both the schema.yaml file and the chart/ folder.

## Test Deployer image locally with mpdev tool

    The dev container **gcr.io/cloud-marketplace-tools/k8s/dev** includes all necessary libraries for development. When run, it forwards the gcloud and kubeconfig settings from the host, enabling commands like gcloud and kubectl to be executed within the container.

    You can create an executable tool called mpdev using the dev container to run commands for diagnosing the environment, installing applications, verifying deployments, and deleting applications. It works by running a container, extracting a script, and setting up commands locally.
    
Key mpdev Commands:

Diagnose Environment:
    
    mpdev doctor
    
Install an Application:
    
    mpdev install --deployer=<YOUR DEPLOYER IMAGE> --parameters=<PARAMETERS>
  
Delete an Application:
    
    kubectl delete application <APPLICATION DEPLOYMENT NAME>

Smoke Test/Verify an Application:

    mpdev verify --deployer=<YOUR DEPLOYER IMAGE>
    How You Can Use mpdev

Create the Executable:

    create bin directory in home directory first

    mkdir -p $HOME/bin
    
    BIN_FILE="$HOME/bin/mpdev"
    docker run gcr.io/cloud-marketplace-tools/k8s/dev cat /scripts/dev > "$BIN_FILE"
    chmod +x "$BIN_FILE"

Run the mpdev Commands: After setting up mpdev, you can run any of the available commands. For example, to install an application:
    
    mpdev install --deployer=gcr.io/your-company/deployer --parameters='{"name": "deployment", "namespace": "ns"}'

    This tool is highly useful for managing applications on Google Cloud Marketplace and automating deployment processes.
    
## Issue daigonies during mpdev installation

Since $HOME/bin is not in your $PATH, you'll need to add it manually. Here's how you can do that:
    
    Add $HOME/bin to your $PATH by modifying your ~/.bashrc file:
    
    echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
   
 Reload your shell configuration so the changes take effect:
    
    source ~/.bashrc

Verify the updated $PATH:
    
Run the following command to ensure that $HOME/bin is now included in your $PATH:
    
    echo $PATH
    
You should see something like this:

    /home/hassan/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
    
Test mpdev:
    
Now, try running the mpdev command again:
    
    mpdev
   
This should resolve the issue! Let me know if it works or if you encounter any further problems.

## Use use mpdev doctor

    The mpdev doctor command has verified your environment and provided a message about your Google Cloud project

it my case it has shown this below and do the nessecary this to make it resolve..

    mpdev doctor
    + KUBE_CONFIG=/home/hassan/.kube/config
    + GCLOUD_CONFIG=/home/hassan/.config/gcloud
    + EXTRA_DOCKER_PARAMS=
    + DOCKER_NETWORK=host
    + MARKETPLACE_TOOLS_TAG=latest
    + MARKETPLACE_TOOLS_IMAGE=gcr.io/cloud-marketplace-tools/k8s/dev
    ++ date +%Y%m%d-%H%M%S
    + VERIFICATION_LOGS_PATH=/home/hassan/.mpdev_logs/20240916-201334
    + kube_mount=
    + [[ -f /home/hassan/.kube/config ]]
    + kube_mount=(--mount "type=bind,source=${KUBE_CONFIG},target=/mount/config/.kube/config,readonly")
    + gcloud_mount=
    + gcloud_original_path=
    + [[ -e /home/hassan/.config/gcloud ]]
    + gcloud_mount=(--mount "type=bind,source=${GCLOUD_CONFIG},target=/mount/config/.config/gcloud,readonly")
    ++ readlink -f /home/hassan/.config/gcloud
    + canonical_gcloud_config=/home/hassan/.config/gcloud
    + gcloud_original_path=(--env "GCLOUD_ORIGINAL_PATH=${canonical_gcloud_config}")
    + terminal_docker_param=-t
    + [[ -t 0 ]]
    + terminal_docker_param=-it
    + mkdir -p /home/hassan/.mpdev_logs/20240916-201334
    + echo 'Logs stored in /home/hassan/.mpdev_logs/20240916-201334'
    + tee /home/hassan/.mpdev_logs/20240916-201334/verify.log
    Logs stored in /home/hassan/.mpdev_logs/20240916-201334
    + tee /home/hassan/.mpdev_logs/20240916-201334/verify.log
    + docker run --init --mount type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock,readonly --mount type=bind,source=/home/hassan/.mpdev_logs/20240916-201334,target=/logs --net=host --mount type=bind,source=/home/hassan/.kube/config,target=/mount/config/.kube/config,readonly --mount type=bind,source=/home/hassan/.config/gcloud,target=/mount/config/.config/gcloud,readonly --env GCLOUD_ORIGINAL_PATH=/home/hassan/.config/gcloud -it --rm gcr.io/cloud-marketplace-tools/k8s/dev:latest doctor
    
    Your gcloud default project: world-learning-400909
    
    You can set an environment variables to record the GCR URL:
    
      export REGISTRY=gcr.io/$(gcloud config get-value project | tr ':' '/')
    
    ====
    
    kubectl is not installed.
    
    See https://kubernetes.io/docs/tasks/tools/install-kubectl/#download-as-part-of-the-google-cloud-sdk
    
    ====

it gave me the information that kubectl is not install. but i have already install it in my system.. I made below changes for resolving this error..

    It looks like the mpdev doctor command is not recognizing kubectl even though it is installed on your host machine. This issue is likely due to the Docker container not having access to the /snap/bin directory where kubectl is installed.
    
    Here’s a step-by-step approach to ensure that the mpdev tool can test the GCP Marketplace image:
    
    1. Verify kubectl Installation
    Ensure kubectl is correctly installed on your system:
    
    which kubectl

    You should see /snap/bin/kubectl or the path where kubectl is installed.
    
    2. Update Docker Path
    
    Since kubectl is installed in /snap/bin, and Docker containers do not typically have access to this directory, you can try creating a symbolic link to a more accessible path or copying kubectl to a directory that the Docker container can access.
    
    Option 1: Create a Symbolic Link
 
    sudo ln -s /snap/bin/kubectl /usr/local/bin/kubectl
    
    Option 2: Copy kubectl to a Common Directory
     
    sudo cp /snap/bin/kubectl /usr/local/bin/
   
     3. Test kubectl Inside Docker
    Run a Docker container and check if kubectl is accessible:

    docker run -it --rm gcr.io/cloud-marketplace-tools/k8s/dev:latest /bin/bash
    
    Once inside the container:

    which kubectl

    now you will find kubectl inside the container.. connect the cluster and test kubectl for getting cluster info. 

    **Run mpdev doctor Again
Exit the container and retry the mpdev doctor command on your host machine to see if it recognizes kubectl now:**
    
    mpdev doctor    

    And i have found that it is successfully recognized 

I have connect the cluster with my local machine and rerun the **mpdev doctor** it is showing that CDR not is being install in your cluster

    mpdev doctor
    + KUBE_CONFIG=/home/hassan/.kube/config
    + GCLOUD_CONFIG=/home/hassan/.config/gcloud
    + EXTRA_DOCKER_PARAMS=
    + DOCKER_NETWORK=host
    + MARKETPLACE_TOOLS_TAG=latest
    + MARKETPLACE_TOOLS_IMAGE=gcr.io/cloud-marketplace-tools/k8s/dev
    ++ date +%Y%m%d-%H%M%S
    + VERIFICATION_LOGS_PATH=/home/hassan/.mpdev_logs/20240916-204551
    + kube_mount=
    + [[ -f /home/hassan/.kube/config ]]
    + kube_mount=(--mount "type=bind,source=${KUBE_CONFIG},target=/mount/config/.kube/config,readonly")
    + gcloud_mount=
    + gcloud_original_path=
    + [[ -e /home/hassan/.config/gcloud ]]
    + gcloud_mount=(--mount "type=bind,source=${GCLOUD_CONFIG},target=/mount/config/.config/gcloud,readonly")
    ++ readlink -f /home/hassan/.config/gcloud
    + canonical_gcloud_config=/home/hassan/.config/gcloud
    + gcloud_original_path=(--env "GCLOUD_ORIGINAL_PATH=${canonical_gcloud_config}")
    + terminal_docker_param=-t
    + [[ -t 0 ]]
    + terminal_docker_param=-it
    + mkdir -p /home/hassan/.mpdev_logs/20240916-204551
    + echo 'Logs stored in /home/hassan/.mpdev_logs/20240916-204551'
    + tee /home/hassan/.mpdev_logs/20240916-204551/verify.log
    Logs stored in /home/hassan/.mpdev_logs/20240916-204551
    + docker run --init --mount type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock,readonly --mount type=bind,source=/home/hassan/.mpdev_logs/20240916-204551,target=/logs --net=host --mount type=bind,source=/home/hassan/.kube/config,target=/mount/config/.kube/config,readonly --mount type=bind,source=/home/hassan/.config/gcloud,target=/mount/config/.config/gcloud,readonly --env GCLOUD_ORIGINAL_PATH=/home/hassan/.config/gcloud -it --rm gcr.io/cloud-marketplace-tools/k8s/dev:latest doctor
    + tee /home/hassan/.mpdev_logs/20240916-204551/verify.log
    
    Your gcloud default project: world-learning-400909
    
    You can set an environment variables to record the GCR URL:
    
      export REGISTRY=gcr.io/$(gcloud config get-value project | tr ':' '/')
    
    ====
    
    Application CRD is not installed in your cluster.
    
    Run the following to install it:
    
    kubectl apply -f "https://raw.githubusercontent.com/GoogleCloudPlatform/marketplace-k8s-app-tools/master/crd/app-crd.yaml"
    
    For more details about the application CRD, see
    https://github.com/kubernetes-sigs/application

Solution:

    kubectl apply -f "https://raw.githubusercontent.com/GoogleCloudPlatform/marketplace-k8s-app-tools/master/crd/app-crd.yaml"

After the i rerun **mpdev doctor** again and it is showing that everything is good to go!!!

    mpdev doctor
    + KUBE_CONFIG=/home/hassan/.kube/config
    + GCLOUD_CONFIG=/home/hassan/.config/gcloud
    + EXTRA_DOCKER_PARAMS=
    + DOCKER_NETWORK=host
    + MARKETPLACE_TOOLS_TAG=latest
    + MARKETPLACE_TOOLS_IMAGE=gcr.io/cloud-marketplace-tools/k8s/dev
    ++ date +%Y%m%d-%H%M%S
    + VERIFICATION_LOGS_PATH=/home/hassan/.mpdev_logs/20240916-204647
    + kube_mount=
    + [[ -f /home/hassan/.kube/config ]]
    + kube_mount=(--mount "type=bind,source=${KUBE_CONFIG},target=/mount/config/.kube/config,readonly")
    + gcloud_mount=
    + gcloud_original_path=
    + [[ -e /home/hassan/.config/gcloud ]]
    + gcloud_mount=(--mount "type=bind,source=${GCLOUD_CONFIG},target=/mount/config/.config/gcloud,readonly")
    ++ readlink -f /home/hassan/.config/gcloud
    + canonical_gcloud_config=/home/hassan/.config/gcloud
    + gcloud_original_path=(--env "GCLOUD_ORIGINAL_PATH=${canonical_gcloud_config}")
    + terminal_docker_param=-t
    + [[ -t 0 ]]
    + terminal_docker_param=-it
    + mkdir -p /home/hassan/.mpdev_logs/20240916-204647
    + echo 'Logs stored in /home/hassan/.mpdev_logs/20240916-204647'
    + tee /home/hassan/.mpdev_logs/20240916-204647/verify.log
    Logs stored in /home/hassan/.mpdev_logs/20240916-204647
    + docker run --init --mount type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock,readonly --mount type=bind,source=/home/hassan/.mpdev_logs/20240916-204647,target=/logs --net=host --mount type=bind,source=/home/hassan/.kube/config,target=/mount/config/.kube/config,readonly --mount type=bind,source=/home/hassan/.config/gcloud,target=/mount/config/.config/gcloud,readonly --env GCLOUD_ORIGINAL_PATH=/home/hassan/.config/gcloud -it --rm gcr.io/cloud-marketplace-tools/k8s/dev:latest doctor
    + tee /home/hassan/.mpdev_logs/20240916-204647/verify.log
    
    Your gcloud default project: world-learning-400909
    
    You can set an environment variables to record the GCR URL:
    
      export REGISTRY=gcr.io/$(gcloud config get-value project | tr ':' '/')
    
    ====
    
    Everything looks good to go!!

## How run deployer image using mpdev install command..

    When deploying the deployer image from Google Cloud Marketplace, you need to provide following variables.
    
    You can obtain the required variable information using the following commands.
    
    GCP project id: Your GCP Project ID
    
    GCP Cloud SQL Db Connection String in Base64 Encoded format
    
    Creating Postgres Connection String
    
    ENCODED_CONN_STRING=$(echo -n "postgresql://postgres:$(gcloud secrets versions access latest --secret=DB_PASSWORD)@$(gcloud secrets versions access latest --secret=DB_HOST)/postgres" | base64 -w 0)
    
    Use below commands for Verificaiton
    
    echo "$ENCODED_CONN_STRING"
    
    echo "ENCODED_CONN_STRING: $(echo "$ENCODED_CONN_STRING" | base64 --decode)"
    
    GKE kube API Server URL in Base64 Encoded format
    
    Fetch private endpoint of Cluster

    echo "INTERNAL_ENDPOINT: $(echo -n "https://$(gcloud container clusters describe disearch-cluster --zone us-central1-c --format="get(privateClusterConfig.privateEndpoint)")" | base64)"
    
        Decode the base64 encoded endpoint for verfication
    
          echo <INTERNAL_ENDPOINT_ENCODED_VALUE> | base64 --decode
      
    - Website URL: CLIENT WEBSITE URL
    - Client Email Address: EMAIL ADDRESS
      
    - DB User
    
        gcloud secrets versions access latest --secret="DB_USER"
     
    - DB Password 
    
        gcloud secrets versions access latest --secret="DB_PASSWORD"
    
    - DB Host
    
        gcloud secrets versions access latest --secret="DB_HOST"
    
    - GCP Bucket 
    
        gcloud secrets versions access latest --secret="GCP_BUCKET"
    
    - Cloudfunction URLs
        Name: 
          document-status:
    
            gcloud functions describe document-status --region=us-central1 --format="value(url)"
    
          image-processing:
    
            gcloud functions describe image-processing --region=us-central1 --format="value(url)"
    
          update_metadata_ingested_document:
    
            gcloud functions describe update_metadata_ingested_document --region=us-central1 --format="value(url)"
    



    mpdev install --deployer=gcr.io/aretecinc-public/disearch/deployer/deployer:1.0 --parameters='{
      "Project ID": "world-learning-400909",
      "cloudSqlDbConn": "cG9zdGdyZXNxbDovL3Bvc3RncmVzOkNLaEVKWkhbdUZTOyU9TWc4aXd1ZVZtJnhAMTAuNTAuMC4zL3Bvc3RncmVz",
      "k8sApiServerUrl": "aHR0cHM6Ly8xMC4zLjAuMg==",
      "allowedOrigin": "worldlearning.dataimagineers.ai",
      "authEmail": "muhammadhassanb122@gmail.com",
      "dbUser": "postgres",
      "dbPassword": "CKhEJZH[uFS;%=Mg8iwueVm&x",
      "dbHost": "10.50.0.3",
      "gcpBucket": "disearch-storage-bucket-kda92b65",
      "openaiApiKey": "",
      "statusCloudFn": "https://us-central1-world-learning-400909.cloudfunctions.net/document-status",
      "imageSummaryCloudFn": "https://us-central1-world-learning-400909.cloudfunctions.net/image-processing",
      "updateMetadataFn": "https://us-central1-world-learning-400909.cloudfunctions.net/update_metadata_ingested_document"
    }'


    




