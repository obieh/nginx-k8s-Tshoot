# Nginx-k8s-Tshoot
This Projects Demonstrates How to Troubleshoot Issues with Nginx Application in Kubernetes.

## Requirements
* Minikube should be up and running
* Basic understanding of kubernetes and how to use kubectl
* Know how to use any text editor, vi, vim or nano.

## Set UP ENV

* Start minikube
* Run `minikube start` to initialise minikube node on your computer system.

![minkube-start](./img/start%20minikube.png)

## Check your cluster status to be sure your minikube node is up and running

* Run `minikube status` 

![minikude status](./img/minikube-status-after-start.png)

## Create Alias for kubectl key. This simply means you use 'k' instead of the full 'kubectl' keyword when running the command.

* run `alias k=kubectl`

![alias](./img/alias.png)

## Export dry run command

* Run `export do="--dry-run=client -o yaml`

![export](./img/export-dry-run.png)

## Breakdown of the command

* export do =  : create an ENV called do
* do stores a value --dry-run=client -o yaml
* You can now type $do instead of --dry-run=client -o yaml
* --dry-run=client means Just prepare it, don't actually do it yet
* -o yaml means Print it out as yaml

## Create namespace 'dev' if not existing

![namespace](./img/Screenshot%20from%202025-04-30%2012-10-17.png)

## Create the web-pod yaml (dry run)
* Run ` k run web-pod --image=nginx --namespace=dev --port=80 $do > pod.yaml`
* This will just create the yaml file and save it as pod.yaml

## View the dry-run yaml file

* Run `cat pod.yaml` to view the file on terminal
![catpod1](./img/Screenshot%20from%202025-04-30%2012-11-02.png)

## Create the pod from yaml

* Run `k apply -f pod.yaml` to create the web-pod.

![apply](./img/1stk-apply.png)


## Check pod status

* To check if the pod is running `k get pods -n dev`

![get](./img/k-get-pods.png)

## TroubleShooting

### The pod web-pod has 'ErrImagePull' error. 

* Run `k describe pod web-pod -n dev` to see more detailed pod status

![desc](./img/describe1.png)

### Status of the pod says 'waiting' and the reason is  **ImagePullBackOff**.
* Run `k logs web-pod -n dev` to view pod logs

![log](./img/k-log1.png)


### The *ImagePullBackOff* error is a common error message in Kubernetes that occurs when a container running in a pod fails to pull the required image from a container registry. This error can occur for a variety of reasons, including network connectivity issues, incorrect image name or tag, missing credentials, or insufficient permissions.
Scroll down to see more information about the error.

![des2](./img/describe2.png)

### Message column says *Failed to pull image "ngninx"...* Looking closely, we see that we have mis spelt nginx to be ngninx.

* Run `cat pod.yaml` to confirm mis spelling.

![cat](./img/cat-afta-1st-error.png)

### Nginx is not correctly spelt as seen from the output of the previous command.

* Use vi text editor to correct it. Run `vi pod.yaml`

![vi](./img/vi-pod.png)

* Change image value under container to **nginx**

![vi](./img/vi-corrected3.png)

* Run `k apply -f pod.yaml` again having corrected the error

![aplly](./img/2nd%20k-apply-error.png)

* Another error **converting YAML to JSON**, It appears we have altered yaml file format.Lets view and format the file again.

![vi](./img/correct-indent.png)

* Move the image indentation back as indicated to look exactly as the pic below

![vi](./img/corrected-indent.png)

* Run `k apply -f pod.yaml` again
![apply](./img/3rd-k-apply-created.png)

### Notice that the command returned *pod/web-pod configured*

### To dig a little deeper as to understand if the app is running as desired.

* Run `k describe pod web-pod -n dev` 

![describe](./img/k-describe-running.png)

### Pod status indicates the pod is up and running, but when you scroll to check for errors, you see that imagepullback error persists.

![desc](./img/kdescribe-afta-creation-still%20imagepullbackoff.png)

### Verify Image name in the pod

* Run `k get pod web-pod -n dev -o=jsonpath="{.spec.containers[*].image}"`

![backoff](./img/k-get-jsonpath.png)

### The image name on pod is correct as the above output indicates.
### Though i ahve corrected the image name but i did not delete the initisl pod so k8s still running the pod with wrong image name.

* To delete existing pod run `k delete pod web-pod -n dev`

![del](./img/pod-delete.png)

* Run `k apply -f pod.yaml` to create the pod again.

![apply-again](./img/k-apply-afta-delete.png)

### The pod have been created successful. Let me run other T-shhot command to ascertain pod is running.

* Run `k get pods -n dev` to see pod details.

![pod](./img/k-get-afta-delete.png)

### Pod appears to be running now.

* Run `k describe pod web-pod -n dev` to see more pod details.

![afta-del](./img/k-describe-afta-delete.png)

### Details in the message column confirm the pod is up and running.

* To view pod logs Run `k get logs web-deb -n dev

![log](./img/k-logs-afta-delete.png)

### Another proof that the is runing well

### Now we can login to the pod bash.

* Run `k exec -it web-pod -n dev --/bin/bash`

![lod](./img/k-exec-afta-delete.png)

### Pod runs successfully.

## Expose nginx Service

### NodePort will be used to expose the service. Only for development purpose.

* Run `kubectl expose pod web-pod --port=80 --type=NodePort -n dev
` to expose the app port
![run-to-expose](./img/nodeportexpose.png)

* Run `minikube service web-pod -n dev --url` to get the url
![expose](./img/expose-node-service.png)

* Copy the entire url port number inclusive to the browser. You should nginx home page.

![home](./img/welcome-nginx.png)

