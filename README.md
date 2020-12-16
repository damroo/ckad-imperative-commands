# CKAD: imperative-commands

### abbreviations
 * po - pods, pod
 * deploy - deployments, deployment
 * netpol - networkpolicy
### Setting up alias and autocompletion with alias 

alias k=kubectl

source <(kubectl completion bash | sed 's/kubectl/k/g')

### View current context
kubectl config current-context

### Core Concepts

#### set image of pod nginx : image name is nginx and correct value is nginx
k set image po nginx nginx=nginx 

K delete po nains -n mynamespace —grace-period=0 —force


k get po nginxtag --v=9 # verbosity of log level —v=0 till —v=9

kubectl get po -o=custom-columns="POD_NAME:.metadata.name, POD_STATUS:.status.containerStatuses[].state"

custom-columns-file

K get po -o=jsonpath=“$.spec.containers[].name”
Jsonpath-file

k get po --sort-by="{.metadata.name}"

k get po --sort-by=.metadata.creationTimestamp


Multi pods===

set expand tab is important before any yanking pasting in vi editor - do it before any editing in vi mode

K logs bb -c bb1 -p
Or
K logs bb -c bb1 —previous

K top po nginx —containers
K top node node01

Under containers:
              ports:
              - containerPort: 80

Q from exam - deployment strategy maxsurge, max unavailable


Pod design ===

k get po --show-labels

K get nodes --show-labels

k run po ng1 --image=nginx --labels=env=prod

Short form has single hyphen, full form double hyphen

k run ng2 --image=nginx -l env=prod

-n=mynamespace for —-namespace=mynamepsace

Q from exam - write events of pod to a txt file

k get po -l env=prod # get pods with label env=prod
OR
k get po --selector=env=prod

k get po --show-labels -l env=prod
OR
k get po --selector=env=dev --show-labels

k get po -l env # get pods with label key as env

k get po -L env # will show all pods with label env. There will be 1 column name env with values under it for each pod

k get po -l 'env in (dev,prod)' # get pods with env = prod and env=dev

k get po --show-labels -l 'env in (dev,prod)'

k label po ng1 env=uat --overwrite # update pod label

k label po ng3 env- # removing a label add minus sign at end

k label po ng{1..3}  app=nginx # add label to  sequential named pods in same command

k get nodes --show-labels # get all nodes with labels

k label node controlplane node=pyaranode

k delete po nginx --grace-period=0 --force # kill pod immideatly

k describe node controlplane | grep -A10 Taint # grep 10 lines after match

k describe po nginx | grep  -i -A5 label
k annotate po nginx name=webapp



k describe po nginx | grep -i -A3 annotation

k annotate po nginx name-           # remove annotation

k delete po --all # delete all pods

K get all —all-namespaces
OR
K get all -A

K get all -n mynamespace
OR
K get all - - namespace=mynamespace

==deployments==

k create deploy webapp --image=nginx --replicas=5

#To get pods of a deployment get by label of deployment

k get po --show-labels -l app=webapp

# to upscale / downscale deployment
k scale deploy webapp --replicas=20

#To get replicates linked with deployment get rs with same label as deployment



k get rs -l app=webapp

-w to watch any command

containers:
   ports:
   - containerPort: 80

#update image in a deployment

kubectl set image deploy webapp nginx=nginx:1.17.4 --record=true

k rollout undo deploy webapp --to-revision=1
kubectl rollout history deploy webapp --revision=7 # info about revision 7. Also node record statement can be stale an incorrect. Verify image in describe.

k rollout pause deploy webapp

K rollout resume deploy webapp # resumes any stuck image update which was paused due to pause command issued earlier

K rollout history deploy webapp

K rollout status deploy webapp

K rollout undo deploy webapp


==hpa = horizontal pod autoscalere==

K get spa

K delete hpa hpa-name

K autoscale deploy webapp —min=10 —max=20 —cpu-percent=85

job===

k create job myjob --image=node -- /bin/sh -c "node -v”.  # to verify node version - node -v in. Job

k delete job --all # —all to delete all

spec.completions
spec.parallelism
spec.backoffLimit
spec. activeDeadlineSeconds

K get jobs -w # w to watch

k create cronjob cjob --image=busybox --schedule="*/1 * * * *" --dry-run=client -o yaml -- /bin/sh -c "date;echo 'hello'"  > cj.yaml
Cj creates a job and pod for every occurrence of scheduled run

===volume===
# hostpath persistent volume
spec:
  hostPath:
    path:

K explain po.spec —recursive | less
 
k explain pv.spec. --recursive | grep -i -A10 hostpath
   hostPath     <Object>
      path      <string>
      type      <string>

Volume outlives container in a pod, but perishes with pod
Persistent volume outlives lifetime of a pod

# Write txt and create a new file- directly /cache should be preexistent
echo  "this is how you do it.." > /cache/abc.txt

OR 
printf  “this is how you write” > /cache/abc.txt
OR 
touch /cache/newfile.txt
echo “txt” > /cache/newfile.txt

Can do a - bash in exec -it mode in container for autocompletion

===config map==

K create cm —from-literal=key1=value1

K get po —-help # use - - help with commands to see examples

K create cm —help # has all examples

Q 109 read 

#Verify env variables of pod

k exec -it nginx -- env

==security context====

kubectl exec -it secbusybox -- sh
id     // it will show the id and group

###secret generic##
Create secret generic mysecret —from-literal=key1=value1

echo -n bXlwYXNzd29yZA== | base64 --decode
Echo -n my password | base64 

# edit base64 value in secret
K edit secret my secret

#Put secret from secret file on file on volume mount

K explain po.spec.containers.livenessProbe


#######Get events of a pod, -o wide, sorted by timestamp

k get events -o wide --sort-by=.metadata.creationTimestamp --field-selector involvedObject.name=busybox

##into a txt file

k get events -o wide --sort-by=.metadata.creationTimestamp --field-selector involvedObject.name=busybox > temp.txt

##get all events into a file sorted by timestamp - cannot get events in -A all namespaces

k get events --sort-by=.metadata.creationTimestamp > bb.txt

Q 139 - revise

K logs -f pod name # -f means follow the log or tail

Q 143 -revise

kubectl top pod --all-namespaces | sort --reverse --key 3 --numeric | head -3 > cpu-usage.txt
#Can be written as for sort part = sort -nrk 3 | head -3 > a.txt

cat /etc/passwd | cut -f 1 -d ":" > /etc/foo/passwd
# cut first column delimited by : to a file

==service

k expose --help. # helpful

 k run temp --image=busybox -it --rm -- /bin/sh     ##temp po

k exec -it  2c -c   2c-b  -n ggckad-s2 -- sh. # to exec in a container as pod has 2 container



Curl -v http://

Sudo - (i)# gives elevated root privilege — brackets not requied
Sudo command # temp root privilege to run command

Ssh <nodename> or IP address of node
Return to base node
Node-1 is main master node of ckad cluster

Set update env using k command, env in pod, sample pod yaml

k run bb --image=busybox --restart=Never --dry-run=client -o yaml --command  -- env > 2po.yaml

# env goes under command in yaml

k run bb --image=busybox --restart=Never --dry-run=client -o yaml  -- env > 2po.yaml
# env goes under args in yaml
 
k run bb --image=busybox --restart=Never --dry-run=client -o yaml  -- /bin/sh -c "env" > 2po.yaml
# args:
     - /bin/sh
     - -c
     - env

kubectl create quota my-quota
--hard=cpu=1,memory=1G,pods=2,services=3,replicationcontrollers=2,resourcequotas=1,secrets=5,persistentvolumeclaims=10

k get po nginx -o jsonpath='{.spec.containers[].image}{"\n"}'

restart=Never with busy box

kubectl run hazelcast --image=hazelcast/hazelcast --env="DNS_DOMAIN=cluster" --env="POD_NAMESPACE=default"

k get po -L app # get labels in a separate column


echo -e "foo3=lili\nfoo4=lele" > file.txt

k run ng8 --image=nginx --limits=cpu=200m,memory=512Mi --requests=cpu=100m,memory=256Mi

alias testbb='k run tempb --image=busybox --restart=Never --rm -it -- /bin/sh'

#Copy from pod folder to local folder
k cp bb4:etc/passwd ./pwd
