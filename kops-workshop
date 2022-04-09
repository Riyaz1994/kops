Provisioning Google Cloud with k8s using it in-house tool, KOPS
-----------------------------------------------------------------------------------------------

****Note: The lines starting with # represents commands

Requirements
...........................
 
<<<<Install KOPS>>>>

#	wget -O kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
#	chmod +x ./kops
#	sudo mv ./kops /usr/local/bin/
# 	kops version
__ __ __ __ __ __ __ __

<<<<Install Gcloud SDK>>>>> - google cloud sdk is preinstalled on GCP Vms 
 
#	gcloud version

__ __ __ __ __ __ __ __


<<<<Install Kubectl>>>>>

#	sudo apt-get update -y
#	sudo apt-get install -y apt-transport-https ca-certificates curl
#	sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
#	echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
#	sudo apt-get update -y
#	sudo apt-get install -y kubectl
#	kubectl version

__ __ __ __ __ __ __ __

<<<<Authenticate gcloud login >>>

#	gcloud init
select option [2]
Copy the command displayed on the screen and paste in the gcloud cli ( Steps given below)

-- -- -- --
<<<<gcloud init>>>>

*Install gcloud cli for windows (you local pc) from https://cloud.google.com/sdk/docs/install
  just follow the screen instructions to install
  once installed open the gcloud cli

*Paste the command in the gcloud cli and authenticate the account access in the browser prompt, return to the gcloud cli and 
  copy the output and paste it in the VM's command prompt 
-- -- -- --- -- 
 <<<<gcloud init>>>> (Alternate option)

*There is an alternate option to authenticate the gcloud login without launching browser, you can run the command in a new terminal

#	gcloud auth login --no-launch-browser

This will genertae a http url which you can paste in your local browser and authenticate the login, a key is genertaed, this has to be
copied and pasted in the VM's command prompt
-- -- -- -- --

*Once the gcloud is authenticated try few gcloud commands

#	gcloud compute instances list
#	gcloud compute networks list
#	gcloud compute networks subnets list
#	gsutil list

__ __ __ __ __ __

*Now create a bucket to store the state of the cluster

#	gsutil mb gs://<bucket-name>/

#	export KOPS_STATE_STORE=gs://<bucketname>/ 
-- --- ---- ----- ----

<<<<<Let's create the cluster>>>>>>

*run the following commands

#	PROJECT=`gcloud config get-value project`
#	export KOPS_FEATURE_FLAGS=AlphaAllowGCE # to unlock the GCE features
#	kops create cluster <cluster-name> --zones <zone-name> --state gs://<bucket-name>/ --project=${PROJECT} --kubernetes-version=1.11.1 --node-count 3

*If you have exported the variable KOPS_STATE_STORE , it is not necessary to mention --state flag in the above command

The above command creates cluster but only be genertaed when the following command is executed

#	kops update cluster <cluster-name> --state gs://<bucket-name> --yes

It takes a minimum of 8 to 10min for the cluster to get running
once the cluster is up rin the following command 

#	kops validate cluster --state gs://<bucket-name>/

run the command 

#	kops export kubecfg --admin

then run 

#	kops validate cluster --state gs://<bucket-name>/ 

**Master and 3 Nodes are up and running

*******Now let's install krew - A plugin manager for kubernetes components
	the script is available in our github repo, link will be provided in the description
	https://github.com/ncodeit-io/devops-cloud/tree/main/kubernetes/scripts

**clone the repo and navigate to devops-cloud/kubernetes/scripts
   run all the 3 scripts one after another
   pay attention to the screen and follow the instructions provided post execution
   export the variables or add them to the .bashrc file
    we have installed 
1. krew - plugin manager
2. kubectx - tool to switch between contexts (multiple clusters) 
3. kubens - tool to switch between namespaces
4. k9s - terminal based UI to interact with K8s cluster

kubectx & Kubens commands

#	kubectl ctx
#	kubectl ns
#	kubectl ns <namespace-name>


To delete the cluster and resources created 

#	kops delete cluster <cluster-name> --state gs://<bucket-name>/ --yes

this command deletes the instances, subnets, firewalls, vpc but does not delete the bucket
we have to delete it using 

#	gsutil rb gs://<bucket-name>/
