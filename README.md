# 100 Days of DevOps with KodeKloud - Days 51 - 60


We finally made it past the halfway point! Lets pick up where we left off and knock out more Kubernetes (K8s) labs!


## Day 60: Persistent Volumes in Kubernetes



## Day 59: Troubleshoot Deployment issues in Kubernetes



## Day 58: Deploy Grafana on Kubernetes Cluster



## Day 57: Print Environment Variables



## Day 56: Deploy Nginx Web Server on Kubernetes Cluster



## Day 55: Kubernetes Sidecar Containers



## Day 54: Kubernetes Shared Volumes



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



