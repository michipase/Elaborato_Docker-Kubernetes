Michele Pasetto VR495361
# Introduzione a Docker & Kubernetes
## Installazione Docker
Dal sito ufficiale [Docker Docs](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository), è stato eseguito il seguente script per installare Docker e tutti i componenti e plugins utili per l'elaborato
```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Install docker engine and essentials
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Test installation
sudo docker run hello-world
```

All'esecuzione del quale, correttamente compare il messaggio dal docker di test "hello-world"

```console
x@y:~/docker-kubernetes-univr$ sudo docker run hello-world
[sudo] password for x: 

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

Questo conferma la corretta installazione di Docker.
```console
gingko@y:~/docker-kubernetes-univr$ docker --version
Docker version 25.0.3, build 4debf41
```

## Minikube setup
### Install minikube
From the [official documentation](https://minikube.sigs.k8s.io/docs/start/), the binary minikube has been download and installed in `/usr/local/bin/minikube`.

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

The minikube can now be started
```console
x@y:~/docker-kubernetes-univr$ minikube start
😄  minikube v1.32.0 on Ubuntu 22.04
✨  Using the virtualbox driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🏃  Updating the running virtualbox "minikube" VM ...
🐳  Preparing Kubernetes v1.28.3 on Docker 24.0.7 ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
    ▪ Using image docker.io/kubernetesui/metrics-scraper:v1.0.8
    ▪ Using image docker.io/kubernetesui/dashboard:v2.7.0
💡  Some dashboard features require the metrics-server addon. To enable all features please run:

        minikube addons enable metrics-server


🌟  Enabled addons: storage-provisioner, default-storageclass, dashboard
💡  kubectl not found. If you need it, try: 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

Also an alias for minikube kubectl can be created using:
```bash
alias kubectl="minikube kubectl --"
```
Keep in mind this alias is only temporary, as it will not be loaded again in the next terminal session. It can be made permanet adding it to your `.bash_aliases` or `.bashrc` which is usually found in home folder.

### Minikube addons
For the purpose of the test, ingress addon has been added to our minikube.
minikube addons enable ingress
```console
x@y:~/docker-kubernetes-univr$ minikube addons enable ingress
💡  ingress is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
    ▪ Using image registry.k8s.io/ingress-nginx/controller:v1.9.4
    ▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20231011-8b53cabe0
    ▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20231011-8b53cabe0
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled
```

### extract minikube IP
The minikube IP can be found with
```console
x@y:~/docker-kubernetes-univr$ minikube ip
192.168.59.100
```

## Exercise
The `Wrap up exercise file` is downloaded
```console
x@y:~/docker-kubernetes-univr$ curl -O https://raw.githubusercontent.com/mmul-it/training/master/Kubernetes-From-Scratch/Kubernetes-From-Scratch-Wrap-Up-Test.yaml
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1316  100  1316    0     0   1531      0 --:--:-- --:--:-- --:--:--  1532
```
Using `sed` the host address for the ingress element can be changed to our minikube IP address
```bash
sed -i -e 's/host: .*/host: 192.168.59.100.nip.io/g' Kubernetes-From-Scratch-Wrap-Up-Test.yaml
```

The yaml file gets loaded into the Minikube cluster

```console
x@y:~/docker-kubernetes-univr$ kubectl create -f Kubernetes-From-Scratch-Wrap-Up-Test.yaml
namespace/myingress-test created
configmap/docroot created
pod/mywebserver created
service/mywebserver-svc created
ingress.networking.k8s.io/mywebserver-ingress created
```

# Test 1

The previous commands will create all the resources specified in the YAML file, and can be check using
```console
x@y:~/docker-kubernetes-univr$ kubectl -n myingress-test get all,ingress
NAME              READY   STATUS    RESTARTS   AGE
pod/mywebserver   1/1     Running   0          98s

NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/mywebserver-svc   ClusterIP   10.108.219.177   <none>        80/TCP    98s

NAME                                            CLASS   HOSTS                   ADDRESS          PORTS   AGE
ingress.networking.k8s.io/mywebserver-ingress   nginx   192.168.59.100.nip.io   192.168.59.100   80      98s
```

The used YAML created a pod called "mywebserver" which allow traffic from port 80 (used by HTTP protocol) which looks like it is served using an nginx proxy.

# Test 2
Since HTTP server is used to serve resurced and usually website, we can use curl to make a GET request to our new server.
```console
x@y:~/docker-kubernetes-univr$ curl http://192.168.59.100.nip.io
This is my Ingress/nginx test
Good job!
``` 