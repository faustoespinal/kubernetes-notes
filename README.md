# kubernetes-notes
Notes on Installing and setting up Kubernetes on vagrant or openstack based systems using coreos as base operating system for worker and master nodes.  These instructions were tried out using a VirtualBox setup and a configured vagrant virtualbox provider.

## Main Links for Setup
* [Vagrant based kubernetes install](https://coreos.com/kubernetes/docs/latest/kubernetes-on-vagrant.html)
* [Setting up HAProxy Ingress Controller](https://github.com/kubernetes/ingress/tree/master/examples/deployment/haproxy)

## Variations to Vagrant-based Install
The kubernetes install creates a virtual private network which is not accesible from outside the host machine.  Vagrant allows for attaching a box to a public network with static IP configuration.  In the main Vagrantfile under **coreos-kubernetes/multi-node/vagrant/Vagrantfile** add:

```
...
      controllerIP = controllerIP(i)
      controller.vm.network :private_network, ip: controllerIP
      controller.vm.network :public_network, ip: "192.168.2.245"
...
```

## Run Kubernetes Web GUI
The web GUI comes configured and ready to go by default with the coreos-vagrant install but it must be executed using kubectl. By default it will only accept connections from internal hosts, in order to allow connections from anywhere additional parameters must be provided. A shell script to accomplish this looks like:

```
#!/bin/bash

echo 'Running Web-UI....'
kubectl proxy --address='0.0.0.0' --accept-hosts='^*$'
```

## Setting up an HAProxy Ingress Controller
Below is an abbreviated set of instructions from the Github HAproxy Ingress controller instructions:

```
openssl req   -x509 -newkey rsa:2048 -nodes -days 365   -keyout tls.key -out tls.crt -subj '/CN=localhost'
kubectl create secret tls tls-secret --cert=tls.crt --key=tls.key

# Explicitly run these pods.
kubectl run http-svc   --image=gcr.io/google_containers/echoserver:1.3   --port=8080   --replicas=1   --expose
kubectl run ingress-default-backend   --image=gcr.io/google_containers/defaultbackend:1.0   --port=8080   --limits=cpu=10m,memory=20Mi   --expose
kubectl get pods

kubectl create configmap haproxy-ingress

# This deploys the actual haproxy pod.
kubectl create -f haproxy-ingress.yaml

# Sample web-app to hit
kubectl create -f simple-web-app.yaml 

kubectl expose deploy/haproxy-ingress --type=NodePort
kubectl get svc/haproxy-ingress -oyaml
kubectl get nodes
curl -i {IP Address of any cluster Node}:{Port # of service}
curl -i {IP Address of any cluster Node}:{Port # of service} -H 'Host: foo.bar'
```


haproxy.yaml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    run: haproxy-ingress
  name: haproxy-ingress
spec:
  replicas: 1
  selector:
    matchLabels:
      run: haproxy-ingress
  template:
    metadata:
      labels:
        run: haproxy-ingress
    spec:
      containers:
      - name: haproxy-ingress
        image: quay.io/jcmoraisjr/haproxy-ingress
        args:
        - --default-backend-service=default/ingress-default-backend
        - --default-ssl-certificate=default/tls-secret
        - --configmap=default/haproxy-ingress
        ports:
        - name: http
          containerPort: 80
        - name: https
          containerPort: 443
        - name: stat
          containerPort: 1936
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
```


simple-web-app.yaml
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: app
spec:
  rules:
  - host: foo.bar
    http:
      paths:
      - path: /
        backend:
          serviceName: http-svc
          servicePort: 8080
 ```



