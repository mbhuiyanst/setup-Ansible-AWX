# setup-Ansible-AWX
Here I will present how to install ansible AWX on ubuntu 24 using kubernetes k3s


####### 1. Update the system ######
sudo apt update
sudo apt upgrade -y

##### 2. Install k3s #####
curl -sfL https://get.k3s.io | sh -

##### 3. Give Non-root User Access to K3s Config ######

sudo chown $USER:$USER /etc/rancher/k3s/k3s.yaml
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

##### 4. Verify Kubernetes Cluster ####

kubectl get nodes
kubectl get pods -A


#### Install Kustomize ####

curl -s https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh | bash
sudo mv kustomize /usr/local/bin/

##### Create Kustomization Directory #####
mkdir awx-deploy
cd awx-deploy


##### Create kustomization.yaml #####

#create file
nano kustomization.yaml



#Add below to the file
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
 - github.com/ansible/awx-operator/config/default?ref=2.19.1



images:
 - name: quay.io/ansible/awx-operator
   newTag: 2.19.1



namespace: awx


###### Apply Kustomize Configuration #####
kubectl apply -k .


##### Verify Operator is Running #####
kubectl get pods -n awx


##### Create AWX Instance #####
nano awx-demo.yaml
 add these lines 
 apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx-demo
spec:
  service_type: nodeport
  nodeport_port: 32000


##### Add Instance to kustomization.yaml #####
 add in resources 

 resources:
  - github.com/ansible/awx-operator/config/default?ref=2.19.1
  - awx-demo.yaml

#####  Reapply Kustomize Configuration #####
kubectl apply -k .


##### Check Pod Status ####

kubectl get pods -n awx


##### View Logs ####

kubectl logs -f deployment/awx-operator-controller-manager -c awx-manager -n awx

##### Retrieve Admin Password #####
kubectl get secret awx-demo-admin-password -n awx -o jsonpath="{.data.password}" | base64 --decode ; echo

  #### Access the AWX Dashboard #####

  http://<server-ip>:32000

  #Username: admin
  #Password: (store or copy ur password from previous step)







