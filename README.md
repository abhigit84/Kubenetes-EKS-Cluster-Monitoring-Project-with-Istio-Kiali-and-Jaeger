# Kubenetes-EKS-Cluster-Monitoring-Project-with-Istio-Kiali-and-Jaeger

Instance Type : t2.medium / t2.large
AMI : AWS Linux
Storage : 30 GB
Region:us-west-2
i Created key pair when creating ec2

1)Install kubectl

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256) kubectl" | sha256sum --check
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client

2)Install eksctl

curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/bin
eksctl version

3)Create role and attach adminisitrator access policy to it.
Update in EC2 this role so that EC2 access eks.(ec2 select-actions-security-modify iam role-select role-update)

4)Create Cluster(for this check in vpc subnets for av.zones in the particular region and select any 2)

eksctl create cluster --name=eksdemo1 --region=us-west-2 --zones=us-west-2a,us-west-2b --without-nodegroup

5)ADD OIDC

eksctl utils associate-iam-oidc-provider --region us-west-2 --cluster eksdemo1 --approve

6)Add Nodes(after creating cluster added nodes in this step as created cluster without nodegroup,also create keypair with ssh-keygen command and add full path of public key in below command like --ssh-public-key=/home/ec2-user/.ssh/id_rsa.pub )

eksctl create nodegroup --cluster=eksdemo1 --region=us-west-2 --name=eksdemo-ng-public --node-type=t2.medium --nodes=2 --nodes-min=2 --nodes-max=4 --node-volume-size=10 --ssh-access --ssh-public-key=/home/ec2-user/.ssh/id_rsa.pub --managed --asg-access --external-dns-access --full-ecr-access --appmesh-access --alb-ingress-access

7)To display the pods

kubectl get pods -n kube-system

8)Install Istio

curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.18.1 TARGET_ARCH=x86_64 sh -

9)Go into the directory

cd istio-1.18.1

The installation directory contains:
● Sample applications in samples/
● The istioctl client binary in the bin/ directory.


SET THE PATH:

export PATH=$PWD/bin:$PATH

10)INSTALL THE ISTIO WITH DEMO PROFILE

istioctl install --set profile=demo -y


11)now run bellow command to run.yaml file

kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.18/samples/bookinfo/platform/kube/bookinfo.yaml

12)To get Service

kubectl get services

13)To check Pods

kubectl get pods

14)To display the output of our deployment

kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"

15)TO INJECT ISTIO AS INIT CONTAINER [ NOW 2 PODS WILL RUN]

kubectl label namespace default istio-injection=enabled
istioctl analyze

16)Delete all the PODS

kubectl get pods
kubectl delete pod details-v1-7c7dbcb4b5-9pbvg productpage-v1-664d44d68d-p9cxj ratings-v1-844796bf85-rjvtw reviews-v1-5cf854487-7s4dq reviews-v2-955b74755-tpblc reviews-v3-797fc48bc9-27c8d
Note : Here we have to delete the pods then only we can see 2 pods are running
kubectl get pods

17)Go to the Specified Folder

cd ./samples/bookinfo/networking/

run this command to execute bookinfo-gateway.yaml

kubectl apply -f bookinfo-gateway.yaml

18)To Display the gateway

kubectl get vs
kubectl get gateway

19)Run bellow Command to get External IP Address

kubectl get svc istio-ingressgateway -n istio-system

a35b36105586b450da0979f568380d2c-1969108211.us-west-2.elb.amazonaws.com

20)Set the ingress IP and ports:

export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')

echo $SECURE_INGRESS_PORT


21)Export INGRESS_HOST(put lb dns from step 19)

export INGRESS_HOST=a35b36105586b450da0979f568380d2c-1969108211.us-west-2.elb.amazonaws.com

export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT

echo $GATEWAY_URL

22)HIT THE BELOW URL in browser which comes as output from below command (http://a35b36105586b450da0979f568380d2c-1969108211.us-west-2.elb.amazonaws.com:80/productpage)

echo "http://$GATEWAY_URL/productpage"

http://a35b36105586b450da0979f568380d2c-1969108211.us-west-2.elb.amazonaws.com:80/productpage

23)KIALI DASHOBAORD [ ALL TOOLS INSTALLATION ]

cd istio-1.18.1/samples/addons(basically u are in networking folder so 2 times cd .. cd .. and then cd addons)
kubectl apply -f .

24)Do Port Forward

Note : OPEN THE SECURITY_GROUP TO ALL TRAFFIC in ec2 

kubectl port-forward --address 0.0.0.0 svc/kiali 9008:20001 -n istio-system

Hit Bellow Command in browser to enter in to Kiali Portal

<public IP Address ec2>:9008 
34.222.32.245:9008

25)FOR JAEGER

kubectl port-forward --address 0.0.0.0 svc/tracing 8008:80 -n istio-system

hit in browser:-
<public IP Address ec2>:8008
34.222.32.245:8008

26)Delete:
DELETE NODE:-
eksctl delete nodegroup --cluster=eksdemo1 --region=us-west-2 --name=eksdemo-ng-public

DELETE CLUSTER:-
eksctl delete cluster --name=eksdemo1 --region=us-west-2

Note :
if Kiali graph does not come up keep hitting Loadbalancer url of application(product page)-http://a35b36105586b450da0979f568380d2c-1969108211.us-west-2.elb.amazonaws.com:80/productpage to resolve.
