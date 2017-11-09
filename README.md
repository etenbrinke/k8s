Ansible Recipes to Install Kubernetes on CloudStack
=====================================

Basic recipes using the ansible cloudstack module to create ssh keys, sec group etc and deploy [Kubernetes](http://kubernetes.io) on [CoreOS](http://coreos.com).

Prerequisites
-------------

You will need python 2.7 or higher with [virtualenv](https://pypi.python.org/pypi/virtualenv)

    $ sudo pip install virtualenv
    
Setup cs
--------

Create a `~/.cloudstack.ini` file with your creds and cloudstack endpoint:

    [cloudstack]
    endpoint = <cloudstackapiendpoint>
    key = <apiaccesskey> 
    secret = <apisecretkey> 
    method = post

We need to use the http POST method to pass the userdata to the coreOS instances.

We can also use variables:

    CLOUDSTACK_ENDPOINT=<cloudstackapiendpoint>
    CLOUDSTACK_KEY=<apiaccesskey>
    CLOUDSTACK_SECRET=<apisecretkey>
    CLOUDSTACK_METHOD=post

On CloudStack server you have to install libselinux-python

    yum install libselinux-python

Clone the repository and setup environment
---------------

This will install [cs](https://github.com/exoscale/cs) and Ansible

    $ git clone https://github.com/apachecloudstack/k8s
    $ cd k8s
    $ python -mvenv .venv
    $ source .venv/bin/activate
    $ pip install -r requirements.txt

Configure Ansible
-----------------

Copy and edit config.yml

    $ cp config.yml-example config.yml
    

Create a Kubernetes cluster
---------------------------

    $ ansible-playbook --extra-vars @config.yml k8s.yml

Some variables can be edited in the `k8s.yml` file.
This will start a Kubernetes master node and a number of compute nodes.
This is all setup via coreOS instances and passing userdata.

Check the tasks and templates in `roles/k8s`

If you retrieve an error during the ssh key copy:

    "msg": "file (/root/.ssh/id_rsa_k8s) is absent, cannot continue",

Please run the Playbook a second time [(related issue)](https://github.com/apachecloudstack/k8s/issues/5)

Install and configure kubectl
-----------------------------

Now you should have a working cluster.

Install kubectl using the following instructions: https://kubernetes.io/docs/tasks/tools/install-kubectl/

Configure your credentials:

    $ kubectl config set-credentials $USER --client-certificate=certificates/client.crt --client-key=certificates/client.key --embed-certs=true --token=$(cat certificates/token.txt)   
    $ kubectl config set-context default/betanl2/kubeuser --cluster=betanl2 --user=$USER

Deploy your first resources
---------------------------

Dashboard:

    # This will create the certificate
    $ kubectl create secret generic kubernetes-dashboard-certs --from-file=certificates/dashboard -n kube-system
    
    # Deploy the dashboard
    $ kubectl apply -f resources/kubernetes-dashboard.yaml

CoreDNS:
    
    $ kubectl apply -f resources/coredns.yaml

Heapster:

    $ kubectl apply -f resources/heapster

Create etcd cluster
-------------------

That's a bonus to this work, there is a playbook to create an independent etcd cluster.

    $ ansible-playbook etcd.yml

Edit some of the variables in the `etcd.yml` file directly.

Important Notice
-------------

If you want to run it on a CloudStack environment with VMware ESX hosts cluster. You have to comment the lines refer to secgroups and secgroup_rules (specific to KVM) :
    k8s.yml
    k8s/tasks/main.yml
    k8s/tasks/create_vm.yml
    etcd.yml
    etcd/tasks/main.yml
