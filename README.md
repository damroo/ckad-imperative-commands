# CKAD: imperative-commands

## Abbreviations
 * po - pods, pod
 * deploy - deployments, deployment
 * netpol - networkpolicy
 * rs - replicaset, replicasets
 * cj - cronjob
 * pv - persistent volume
 * pvc - persistent volume claim
 
## Setting up alias and autocompletion with alias 

alias k=kubectl

source <(kubectl completion bash | sed 's/kubectl/k/g')

## View current context
kubectl config current-context

## Core Concepts

### Set image of pod nginx : image name is nginx and correct value is nginx
k set image po nginx nginx=nginx 

### Force delete a pod
K delete po nains -n mynamespace —grace-period=0 —force

### Verbosity level of information: level —v=0 till —v=9
k get po nginxtag --v=9 

### Get custom columns in pod summary. Notation is similar to jsonpath notation. File can also be used with custom-columns-file
kubectl get po -o=custom-columns="POD_NAME:.metadata.name, POD_STATUS:.status.containerStatuses[].state"


### Get pod field details using jsonpath notation. File can also be used with jsonpath-file
K get po -o=jsonpath=“$.spec.containers[].name”

### Sort pod summary by name
k get po --sort-by="{.metadata.name}"

### Sort pod summary by timestamp
k get po --sort-by=.metadata.creationTimestamp


## Logging and Debugging

### Get log dump of a pod
k logs <pod-name> 

### Tail logs of a pod or follow logs
k logs -f <pod-name> 

### Get logs of a container in multi container pod
k logs <pod-name> -c <container-name>

### Get previous logs of a container
k logs <pod-name> -c <container-name> —p

### Node and pod meterics. Below commands only run if metric server is installed.
k top po nginx —containers
k top node node01

## Labels and selectors

### Get pods summary with labels
k get po --show-labels

### Get node summary with labels
k get nodes --show-labels

### Create a pod using container image nginx and add label while creation
k run po nginx --image=nginx --labels=env=prod

### Another way to add label during pod creation using -l
k run nginx --image=nginx -l env=prod

### Filter pods with specific label name. Get pods with label env=prod
k get po -l env=prod 
OR
k get po --selector=env=prod
OR
k get po --show-labels -l env=prod
OR
k get po --selector=env=dev --show-labels

### Get pods with label whose key name is 'env'

k get po -l env 

###  How all pods with label env. There should be 1 column name 'env' with corresponding values under it for each pod.

k get po -L env 

### Filter pod with more than 1 label values. Get pods with env = prod and env=dev

k get po --show-labels -l 'env in (dev,prod)'

### Update a label value 
k label po ng1 env=uat --overwrite

### Removing a label. Note hyphen at end
k label po ng3 env- 

### Add label to  sequentially named pods in same command
k label po ng{1..3}  app=nginx 

### Label an existing node
k label node controlplane labelname=labelvalue

### Get taint on a node
k describe node controlplane | grep -A3 Taint # grep 3 lines after match

### Annotate a pod. Similar syntax to label.
k annotate po nginx name=webapp

### Verify pod annoation. 
k describe po nginx | grep -i -A3 annotation

### Remove an annotation
k annotate po nginx name-         

### Delete all pods
k delete po --all # delete all pods

### Get all pods across all namespaces
K get all —all-namespaces
OR
K get all -A

### Get all pods in a specific namespace
K get all -n mynamespace
OR
K get all --namespace=mynamespace


## Deployments

### Create deployment with a pod with image as nginx. Replicas = 5

k create deploy webapp --image=nginx --replicas=5

### To get pods of a deployment can fetch by label of deployment

k get po --show-labels -l app=webapp

### Upscale / downscale an existing deployment
k scale deploy webapp --replicas=20

### To get replicates linked with deployment get rs with same label as deployment
k get rs -l app=webapp

### To watch a pod creation in action use -w to watch
k get po -w

### Update image in an existing deployment
kubectl set image deploy webapp nginx=nginx:1.17.4

### Update image and record the command so that it is visible under 'Cause' field in rollout history

kubectl set image deploy webapp nginx=nginx:1.17.4 --record=true

### Undo a deployment
k rollout undo deploy webapp

### Undo a deployment to a specific revision number
k rollout undo deploy webapp --to-revision=1

### Get rollout history of a deployment
kubectl rollout history deploy webap 

### Get history of a specific rollout revision
kubectl rollout history deploy webapp --revision=7

### Pause a rollout
k rollout pause deploy webapp

### Resumes any stuck image update which was paused due to pause command issued earlier
k rollout resume deploy webapp 

### Deployment rollout status
k rollout status deploy webapp

### HPA - horizontal pod autoscaler

k get hpa

k delete hpa <hpa-name>

k autoscale deploy webapp —min=10 —max=20 —cpu-percent=85

## Jobs and Cronjobs

### Create a job which runs a command to verify node version

k create job myjob --image=node -- /bin/sh -c "node -v”

### Delete all jobs
k delete job --all 

### watch jobs summary
k get jobs -w 

### Create cronjob with a scheduled run of every minute
k create cronjob cjob --image=busybox --schedule="*/1 * * * *"  -- /bin/sh -c "date;echo 'hello'"


### Get info on command usage

### Which fields are under spec section of pod in yaml
k explain po.spec —recursive | less

k explain pv.spec. --recursive | grep -i -A10 hostpath
   hostPath     <Object>
      path      <string>
      type      <string>

k get po --help



### Config Map

#### To see all example usages of config map create command
k create cm —help 

k create cm —from-literal=key1=value1


### Verify env variables of pod
k exec -it nginx -- env

### Verify Security Context of a container
kubectl exec -it secbusybox -- sh
id     // it will show the id and group

### Secret
k create secret generic mysecret —from-literal=key1=value1

### Decode a secret
echo -n bXlwYXNzd29yZA== | base64 --decode

### Encode a secret
echo -n mypassword | base64 

### Edit base64 value in secret
k edit secret mysecret



### Get events of a pod, -o wide, sorted by timestamp

k get events -o wide --sort-by=.metadata.creationTimestamp --field-selector involvedObject.name=busybox

### Write above summary into a txt file

k get events -o wide --sort-by=.metadata.creationTimestamp --field-selector involvedObject.name=busybox > temp.txt

### Get all pod events into a file sorted by timestamp

k get events --sort-by=.metadata.creationTimestamp > bb.txt

### Get pods with sorted cpu usage descending order and write top 3 entries to a text file

kubectl top pod --all-namespaces | sort --reverse --key 3 --numeric | head -3 > cpu-usage.txt
OR
kubectl top pod --all-namespaces | sort -nrk 3 | head -3 > a.txt



## Service

### Check usages of expose command
k expose --help


### Execute shell in a container when pod has 2 containers
k exec -it  2c -c   2c-b  -n ggckad-s2 -- sh

## Resource Quota

### Create resource quota
kubectl create quota my-quota
--hard=cpu=1,memory=1G,pods=2,services=3,replicationcontrollers=2,resourcequotas=1,secrets=5,persistentvolumeclaims=10

### Create pod and set env variables during creation
kubectl run hazelcast --image=hazelcast/hazelcast --env="DNS_DOMAIN=cluster" --env="POD_NAMESPACE=default"

### Specify POD resource request and limit during pod creation
k run ng8 --image=nginx --limits=cpu=200m,memory=512Mi --requests=cpu=100m,memory=256Mi



### Copy from pod folder to local folder
k cp <pod-name>:etc/passwd ./pwd


