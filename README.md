# 100 Days of DevOps with KodeKloud - Days 51 - 60


We finally made it past the halfway point! Lets pick up where we left off and knock out more Kubernetes (K8s) labs!


## Day 60: Persistent Volumes in Kubernetes

Well, we're working with Persistent Volumes and Persistent Volume Claims. I haven't worked with either in K8s although I familiar with persistent volumes and mounts in the traditional Linux sense (s/o the RHCSA exam that I passed). Here's my best shot with configuring everything based on the lab prompt. I Frankensteined all of these different manifest type templates together. I don't know if I need to put these Persistent Volume blocks into a YAML file and also apply them. I'll ask Copilot for that part. The updates will be below. 

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-nautilus
  labels:
    app: nautilus-web

spec:
  containers:
    - name: container-nautilus
      image: httpd:latest

      ports:
        - containerPort: 80

      volumeMounts:
        - name: nautilus-storage
          mountPath: /usr/local/apache2/htdocs

  volumes:
    - name: nautilus-storage
      persistentVolumeClaim:
        claimName: pvc-nautilus

```


```
apiVersion: v1
kind: Service
metadata:
  name: web-nautilus

spec:
  type: NodePort

  selector:
    app: nautilus-web

  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30008
```

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nautilus

spec:
  storageClassName: manual

  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 3Gi
```

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nautilus
spec:
  storageClassName: manual
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

 UPDATE: Alright I was pretty spot on with the PersistentVolumeClaim (PVC) block. I just needed to added the resources section and make sure I requested the storage instead of saying the storage that I "had". My document root path was wrong as well but I'm not an Apache expert so I don't care about getting this wrong. My PersistentVolume (PV) block was actually correct! Nice. As far as my service manifest was concerned, it was almost perfect but I removed the selector line from it. I removed it because I didn't think we needed it and I wasn't sure what to put there. Guess we need to add it back. 

 Okay once that's done, we actually do need to create separate files for all of these an apply them chronological starting from PV, PVC, Pod, then Service. 

 You can also use `kubectl get pv` and `kubectl get pvc` to verify that your volumes were created and claimed. I did a `curl -i http://<NodePortIP>:80` to verify that Apache was serving a webpage. I did get an HTTP 200 OK message and some HTML output. I think we're done. 

 Green check mark! Done with the 50s. Onto the 60s!

## Day 59: Troubleshoot Deployment issues in Kubernetes

Dont you just love troubleshooting? If my networking days have told me anything, it's to always check the logs when an issue occurs. There's redis app that went down for this lab so I immediately did all of the kubectl get and describe commands for the pods, deployment, svc, etc. I actually even did it for the ConfigMap as well. I noticed a few issues. The pod was still in a ContainerCreating status. The image name was wrong `redis:alpin` instead of alpine. The configmap had a type "redis-cofig` instead of config. Now, I would try to fix this by going into the manifest files but there are none. So it seems like we need to do some type of rolling update to these things. I'll ask Copilot for this part. 

UPDATE: Copilot pretty much confirmed everything I said. It was the config map typo and the image typo. It gave me the command `kubectl edit deployment redis-deployment` to edit a deployment file which was not listed on my Linux machine. I easily fixed the file and the deployment automatically updated. That was a real confidence booster for me! I feel like I know what I'm doing. Very nice. 

## Day 58: Deploy Grafana on Kubernetes Cluster

We need to make a deployment named `grafana-deployment-xfusion` using any Grafana image for the app. We also need to create a NodePort type service with nodePort 32000 to expose the app. I'm going to take the usual approach and look up a Deployment and Service manifest file. 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana-deployment-xfusion
  labels:
    app: grafana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana-container
        image: grafana/grafana:latest
        ports:
        - containerPort: 3000

```


```
apiVersion: v1
kind: Service
metadata:
  name: grafana-service-xfusion
spec:
  type: NodePort
  selector:
    app: grafana
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
```   

We'll see what Copilot has to say about it. 

UPDATE: Okay I was still pretty close. My image path was wrong. It was originally `grafana:latest` when I should've been `grafana/grafana:latest`. Apparently Grafana listens on port 3000 by default so I needed to change my deployment manifest from port 32000 to port 3000. I'm assuming this is where I change my port to 3000 and my targetPort to 32000 accordingly to the lab. Okay I was close. In my service manifest, the port and targetPort have to be 3000 and I needed to add the nodePort line and make that equal to 32000. 

Run the usual `kubectl aaply -f` on both files to get the Grafana container up and running. It wasn't working at first but I had to wait a second for the dashboard to load. I'm also adding `kubectl get endpoints` to my troubleshooting as well. The `kubectl get logs` had a lot of output but I eventually learn how to decipher through everything. 

On to day 59.

## Day 57: Print Environment Variables

This lab is all about environment variables which I never use. I see why they're important but I never really know when and how to use them. We need to create a pod named `print-envars-greetings` with a container named `print-env-container` that uses the bash image (is there a specific tag for this?). The container will have 3 environment variables named GREETING, COMPANY, and GROUP. We need to set the restartPolicy to Never and use an echo command that will print the 3 environment varibales. The `kubectl logs -f print-envars-greeting` can verify our output. I'm going to attempt this myself before AI. I found this Pod manifest file on the K8s website:

```
apiVersion: v1
kind: Pod

metadata:
  name: print-envars-greeting

spec:
  containers:
    - name: print-env-container
      image: bash

      env:
        - name: GREETING
          value: "Welcome to"

        - name: COMPANY
          value: "DevOps"

        - name: GROUP
          value: "Datacenter"

      command: ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']

  restartPolicy: Never
```

UPDATE: So apparently that was REALLY close. I just needed to remove the template line because that's for deployments and not for Pod manifest files. Once again, I really need to learn each part of the manifest file so I don't run into this issue anymore. I guess the next part is I have to format this because I'm sure the YAML indents are off now since I removed the template line. Lastly, apparently the image should just be bash and not have a tag. Okay, I actually did a pretty good job at this. Going to use the code Copilot gives me instead. 

## Day 56: Deploy Nginx Web Server on Kubernetes Cluster

We have to create an NGINX static webpage deployment with multiple replicas. We'll use the nginx:latest image and the deployment will be named nginx-deployment. The container will be named nginx-container and we'll have 3 replicas. We also have to create  NodePort type service named nginx-service on port 30011. So my first thought is to create a Deployment manifest file. I'm not totally sure about the NodePort type service though. That could be a service file so I'll have to look that up. I'm going to ask google for a generic deployment and service manifest files and then fill in the information from there. Then, we'll use `kubectl apply -f` on each file. 

You can find a generic deployment template on the kubernetes deployments webpage. Same with the service file. I tried to do this myself but it didn't work. Apparently the webpage needs to be up and running. I didn't do a curl test to check this. 

UPDATE: Okay my target port was wrong in my service file. I also think maybe the app: fields didn't match in both of my files either. Other than that, everything was spot on. I may or may not write out of some of these deployment files. I need to understand them completely. Anyway, lab completed!

## Day 55: Kubernetes Sidecar Containers

We're creating an NGINX container with a logging container as a sidecar. I've seen this concept often on Twitter so lets implement it. I altered the Pod manifest file from the previous lab. I completed as much of it as I could but I eventually had to ask AI for help. Here's the final result:

```
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
```

Lets go over the design of the sidecar container. We used the `initContainer` block because the init container runs before the regular application container. So it would start before the NGINX container. It would perform its setup work and then exit. Afterwards, the main container starts. 

The init container doesn't finish because we included the `restartPolicy: Always` line in the block. This turns the sidecar into a restartable init container. This allows it to remain running. 

As far as the commands are concerned, we just need to put them on separate lines and remove the double quotation marks from around them. 

That completes this lab. I'll be going over this lab a few times to get it under my fingers. Defiinitely a learning experience. 


## Day 54: Kubernetes Shared Volumes

This lab was a little more difficult. We needed to create a Pod manifest file that had two containers that used a shared volume. We needed to run a sleep command for them so that they would remain in a running state. We then needed to create a file on one of the containers and make sure we saw that file on the other container although the file paths were differnet. 

I asked AI to help me with this. We first created a Pod manifest file:

```
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
```

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



