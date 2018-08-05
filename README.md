# Install spinnaker in GKE using terraform and helm

### Prerequisite 

* Runs only on Linux or Mac
* Following tools needs to be locally installed and available via `PATH` 
    * [gcloud SDK](https://cloud.google.com/sdk/install)
    * [kubectl](https://cloud.google.com/kubernetes-engine/docs/quickstart) (`gcloud components install kubectl`)
    * [terraform](https://www.terraform.io/intro/getting-started/install.html)
    * [helm CLI](https://docs.helm.sh/using_helm/#installing-helm)
    
### Variables
* `credential`: Contents of a file that contains terraform service account private key in JSON format.
   Default filename is account.json. Refer the below installation steps to generate this file
* `project`: The ID of the GCP project
* `zone`: [Compute engine zone](https://cloud.google.com/compute/docs/regions-zones/) where GKE cluster needs to be created 
* `gcs_location`: Cloud storage [bucket location](https://cloud.google.com/storage/docs/bucket-locations) for storing spinnaker data
    > By default [Nearline](https://cloud.google.com/storage/docs/storage-classes#nearline) storage class is configured. 
    Ensure the correct location based on the configured `zone` 


## Installation steps

1.  `terraform` uses [Service Usage API](https://github.com/terraform-providers/terraform-provider-google/blob/master/CHANGELOG.md#1130-may-24-2018),
    this API needs to be enabled manually
    https://console.cloud.google.com/apis/api/serviceusage.googleapis.com

2.  Create service account for `terraform` and download the key
    ```
    gcloud iam service-accounts create terraform --display-name "terraform"
    gcloud iam service-accounts keys create account.json --iam-account terraform@$(gcloud info --format='value(config.project)').iam.gserviceaccount.com
    ```
    Above command will download the key and store it in `account.json` file
    
3.  Execute below commands. This will take some time to complete (5 to 8 mins)
    ```
    terraform init
    terraform plan -out terraform.plan
    terraform apply terraform.plan 
    ```
    
4.  After the command completes, run the following command to set up port forwarding to the Spinnaker UI from Cloud Shell:
    ```
    export KUBECONFIG=$PWD/.kubeconfig 
    export DECK_POD=$(kubectl get pods --namespace default -l "component=deck" -o jsonpath="{.items[0].metadata.name}")
    kubectl port-forward --namespace default $DECK_POD 8080:9000 >> /dev/null &
    ```
    
5.  Access spinnaker UI at http://localhost:8080/ 
