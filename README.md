# 100 Days of DevOps with KodeKloud - Days 51 - 60


We finally made it past the halfway point! Lets pick up where we left off and knock out more Kubernetes (K8s) labs!


## Day 60: Persistent Volumes in Kubernetes



## Day 59: Troubleshoot Deployment issues in Kubernetes



## Day 58: Deploy Grafana on Kubernetes Cluster



## Day 57: Print Environment Variables



## Day 56: Deploy Nginx Web Server on Kubernetes Cluster



## Day 55: Kubernetes Sidecar Containers

We're creating an NGINX container with a logging container as a sidecar. I've seen this concept often on Twitter so lets implement it. I altered the Pod manifest file from the previous lab. I completed as much of it as I could but I eventually had to ask AI for help. Here's the final result:

apiVersion: v1
kind: Pod
metadata:
  name: webserver
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx

  initContainers:
    - name: sidecar-container
      image: ubuntu:latest
      restartPolicy: Always
      command:
        - sh
        - -c
        - while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep 30; done
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx

  volumes:
    - name: shared-logs
      emptyDir: {}


Lets go over the design of the sidecar container. We used the `initContainer` block because the init container runs before the regular application container. So it would start before the NGINX container. It would perform its setup work and then exit. Afterwards, the main container starts. 

The init container doesn't finish because we included the `restartPolicy: Always` line in the block. This turns the sidecar into a restartable init container. This allows it to remain running. 

As far as the commands are concerned, we just need to put them on separate lines and remove the double quotation marks from around them. 

That completes this lab. I'll be going over this lab a few times to get it under my fingers. Defiinitely a learning experience. 


## Day 54: Kubernetes Shared Volumes

This lab was a little more difficult. We needed to create a Pod manifest file that had two containers that used a shared volume. We needed to run a sleep command for them so that they would remain in a running state. We then needed to create a file on one of the containers and make sure we saw that file on the other container although the file paths were differnet. 

I asked AI to help me with this. We first created a Pod manifest file:

apiVersion: v1
kind: Pod
metadata:
  name: volume-share-nautilus
spec:
  containers:
    - name: volume-container-nautilus-1
      image: debian:latest
      command:
        - sleep
        - "3600"
      volumeMounts:
        - name: volume-share
          mountPath: /tmp/beta

    - name: volume-container-nautilus-2
      image: debian:latest
      command:
        - sleep
        - "3600"
      volumeMounts:
        - name: volume-share
          mountPath: /tmp/apps

  volumes:
    - name: volume-share
      emptyDir: {}

Lets break down this manifest file. The Pod is named `volume-share-nautilus`. You can see the names of both containers. To use a shared volume, you determine where to mount the volume under volumeMounts: and the name must match on both containers. The mountPath does not need to match. The containers will see the same files at their respective destinations. You then define this shared volume under the volumes: section. Name the volume. `emptyDir: {}` creates temporary storage for the lifetime of the Pod. The data remains available if an individual container restarts, but it is deleted when the Pod is deleted.

As far as the sleep command part, here's what AI gave me: "Without the sleep command, a basic Debian container would likely start and immediately exit because it has no long-running process."

We then deployed the Pod using `kubectl apply -f <file_path` and then you can use `kubectl get pods <pod_name>` to verify it has been created. Now, we needed to create the file on container one which we could log into the container using `kubectl exec -it <pod_name> -c <container_name> -- /bin/bash`. Go create the file in the proper place and `cat` to make sure the output is right. Then, log into the second container using the same command above with the updated flag values. 



## Day 53: Resolve VolumeMounts Issue in Kubernetes

Alrighr! I've never done this before so I'm going to look at the prompt and try to figure somethnig out. I'll most likely lean on AI for this but my first thoughs are to at least get the logs using the dscribe commands for both the nodes and pods (and deployment). After doing a `kubectl describe pods`, I see the mounts are defined here.. The `/usr/share/nginx/html` mount is rw. The other mount is ro (looks secrets related). We also have an NGINX container and a PHP-FPM container. The NGINX container has an additional mount. I don't see any issues in the Events. The lab is pointing to the configmap. Now, I haven't had to work with these yet. I'm just going to ask AI for help. 

UPDATE: So apparently the containers mount the emptyDir volume at different locations. They need to see the files in the same place. I was unaware of this. So we needed to get the php-fpm container to have the same file path as the nginx container which was `/var/www/html`. So to recreate the pod exactly how it should be just with a new mount path, we exported the Pod manifext file into a temporary file. We edited this manifest file with the correct file path on the `mountPath:` line. We also removed the uid, resourceVersion, and creationTimestamp lines completely. We then recreated the pod using this new file using `kubectl apply -f`. Then, we copied the PHP file into the Nginx container. I don't know what this step was for. Lastly, we verified the PHP file was present on both containers under their `/var/www/html/index.php` file paths. 

Definitely learned something new. I don't always know how to solve these things but it's reassuring when I know I can use AI to help get the job done. Utilize all resources. That's what makes a good engineer. 

## Day 52: Revert Deployment to Previous Version in Kubernetes

Our prompt asks us to rollback our revisions to the previous version. I've actually never done this before. My first instinct was to check the deployment using kubectl describe to see what the current image is. I see that it was changed from some previous version to nginx:alpine. After speaking with AI, you can check the rollout history using `kubectl rollout history deployment/nginx-deployment`. You can then inspect each revision using the `--revision <#>` flag. There were only two revisions so for the first one, I found that the original image was nginx:1.16. 

We can use the `kubectl rollout undo deployment/nginx-deployment` command to go back one revision. The `--to-revision` flag allows you to choose a specific revision. After running the command, check the rollout status using `kubectl rollout status`. Use the describe option to check the current image. Once everything looks good, we'll submit the lab. 

Got the green check. Lets move to the next one. 

## Day 51: Execute Rolling Updates in Kubernetes

Okay lets kick off the first day of our 50s with K8s rolling updates! I was just doing this yesterday with the kubectl rollout command I believe. We need to use an ipdated image and tag for the deployment named nginx-deployment. We'll verify that the pods are running afterwards. 

Lets start off by getting the current deployment using `kubectl get deployments nginx-deployment`. You can also use `kubectl describe` to get the more detail version. I see with the describe option, we're running image nginx:1.16. Lets update that. We'll use the command `kubectl set image deployment/nginx-deployment nginx-container=nginx:1.19` to update the image. You can use the command `kubectl rollout status deployment/nginx-deployment` to see the rollout status of the deployment. Lastly, use the get and describe options for the deployment and pods to see that the image has been updated. 

Got the green check! That completes this lab. 

Personal Notes:
- kubectl set image is an imperative command that changes the container image in an existing Kubernetes workload. For a Deployment, that changes its Pod template, which automatically triggers a rolling update. We also could've updated Hthe K8s manifest file and hit apply. This is the better production option. 



