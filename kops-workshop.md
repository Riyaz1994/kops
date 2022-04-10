# **Provisioning Google Cloud with k8s using its in-house tool: "KOPS"**
___
* KOPS here stands for *Kubernetes Operations*
* This tool is used to deploy the Cluster over Google Cloud Platform (GCP) or Amazon Web Services (AWS), In this document we will be performing our exercise on GCP
## **Requirements**
___
* ## **KOPS**
* Install kops
 ```
$ wget -O kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64

$ chmod +x ./kops

$ sudo mv ./kops /usr/local/bin/
   
$ kops version
```

* ## **Google cloud SDK**
    * instructions to install google-cloud-sdk are available [here](https://cloud.google.com/sdk/docs/install#deb)
    * Since we are working on GCP VM, Google cloud SDK comes pre-installed, Verify using 
```
$ gcloud version
```

* ## **Kubectl**
* install kubectl
```
$ sudo apt-get update -y

$ sudo apt-get install -y apt-transport-https ca-certificates curl

$ sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

$ echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

$ sudo apt-get update -y

$ sudo apt-get install -y kubectl

$ kubectl version
```

* ## Authenticate gcloud login
```
$ gcloud auth login --no-launch-browser

```

* You will be prompted with the below line

    Do you want to continue (Y/n)? 

    Provide **y** as the option

A url will be generated, go to the following link in your browser: looks similar to this, *the url shown below is only for reference* 

```
https://accounts.google.com/ <your account authentication url >

```
* Click on the url, it lands a page on your browser, authenticate your GCP account, it generates an **Authorization code**, Copy the code and enter in the Command prompt. 
    * ####    **Once the gcloud is authenticated try a few gcloud commands**
```
$ gcloud compute instances list     # to show instances in the project
$ gcloud compute networks list      # lists VPCs
$ gcloud compute networks subnets list  # lists Subnets
$ gsutil list

```
___
* Since installing and configuring the requirements is done, 

**Let's Begin**

**NOTE:**  
Every time you create a cluster, it also creates a Virtual Private Cloud (VPC), per se. 
Google Cloud allows you to create only a maximum of 5 VPC’s in one project, and a total of only 5 clusters

* ## **Create a Bucket**

    * Kops needs a State Store to hold the configuration of our cluster. In our case, it is Google Cloud Storage Buckets. So, let’s create one empty Bucket using the following:
```
$ gsutil mb gs://mykops-bucket/
```
*
    * name of the bucket must be unique
    * Now, since we are ready with the Bucket, we can populate it with our cluster’s State Store
```
$ export KOPS_STATE_STORE=gs://mykops-bucket/ 
```

* ## **Create the Cluster & InstanceGroup Objects in Our State Store**

```
$ PROJECT=`gcloud config get-value project`

$ export KOPS_FEATURE_FLAGS=AlphaAllowGCE # to unlock the GCE features

$ kops create cluster mykops.k8s.local --zones us-central1-a --project=${PROJECT} --kubernetes-version=1.23.5 --node-count 3
```

* Now we can list the Cluster objects in our kops State Store

```
$ kops get cluster
```

* ## **Create a Cluster**

    * We are now ready with all of the changes and the cluster configuration, so we will proceed with the creation of the cluster. 

    * *kops create cluster* created the Cluster object and the InstanceGroup object in our State Store, but it did not actually create any instances or other cloud objects in GCE. To do that, we’ll use *kops update cluster*.
```
    $ kops update cluster mykops.k8s.local --yes
```
*
    * This will take a minimum of 8 to 10 minutes to create all the objects.
    * We are now ready with the cluster, but is it ready for the deployments?
    * Once the kops is finished creating the cluster, we can validate its readiness using the following:
```
$ kops export kubecfg --admin

$ kops validate cluster 
```

* If you find that the cluster not ready, wait for a few minutes as it takes some time to configure the cluster. 
* You can even check using kubectl from your control machine:
```
$ kubectl get nodes
```
* You will see the node counts once your Cluster is up
* verify 1 Master and 3 Nodes are up and running
___
## Now let's install **krew** - A plugin manager for kubernetes components

* The script is available in our github repo click [here](https://github.com/ncodeit-io/devops-cloud/tree/main/kubernetes/scripts
)
* clone the repo on your control machine and navigate to devops-cloud/kubernetes/scripts
```
$ git clone https://github.com/ncodeit-io/devops-cloud.git
$ cd devops-cloud/kubernetes/scripts
```

* find the 3scripts

    * 01-krew-install.sh
    * 02-kubectx-kubens-install.sh
    * 03-k9s-install.sh

* run the script 01-krew-install.sh
```
$ ./01-krew-install.sh 
```
* pay attention to the screen and follow the instructions provided post execution
export the variables or add them to the .bashrc file
```
$ export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
```

* Now execute the second script 02-kubectx-kubens-install.sh
```
$ ./02-kubectx-kubens-install.sh
```
* This will install kubectx and kubens
    * **kubectx** - tool to switch between contexts (multiple clusters) 
    * **kubens** - tool to switch between namespaces

* Now run the third script 03-k9s-install.sh
```
$ ./03-k9s-install.sh

$ export PATH="/home/riyaz_md94/.local/bin:$PATH"
```
* This will install k9s
    * **K9S** - terminal based UI to interact with K8s cluster


* ## kubectx, Kubens and K9s commands to try
```
$ kubectl ctx
$ kubectl ns
$ kubectl ns <namespace-name>
$ k9s
```
___

## **Delete the cluster and resources created**
```
$ kops delete cluster <cluster-name> --state gs://<bucket-name>/ --yes

$ gsutil rb gs://<bucket-name>/
```
___
___
