# kubernetes-notes
Notes on Installing and setting up Kubernetes on vagrant or openstack based systems using coreos as base operating system for worker and master nodes.

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


