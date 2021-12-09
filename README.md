# Installation-recipe-of-OSM-and-microk8s-cluster
This is a recipe to install osm and microk8s using virtualbox. Each deployment is runned in separated virtual machines to avoid the complexity of using two kubernetes backends  (microk8s is not available when kubernetes is working behind)
Two ovas must be downloaded. First one will hold osm engine and the second one microk8s cluster. There will be a conneciton between both thorugh an kubernetes configuration script, that will point microk8s cluster for the osm engine to connect it

The first ova has osm modules installed. Once the virtual machine is deployed, it must be waited for some minutes to start every module. Just type next command to see when the whole system is deployed.
```
 watch kubectl get pods -n osm
 ```
 Once the deployment is checked, the interface of the host-only network must be forced to be able through next order:
 
 ```
 sudo dhclient eth1
```

The first machine is installed and available to work; next step consists of installing microk8s in the other machine. With a simple snap installation command this can be achieved:

```
sudo snap install microk8s --classic --channel=1.21
```

If it was considered to install a multi-node cluster, it would be necessary to deploy more machine with the same ova and insall microk8s in these machines.

Now it is time to give permissions to the master node to microk8s to be used. The command required are the following: 
```
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
su - $USER (password: 'xxx')
```

Also, the interface of host-ony network must be forced as it was forced in osm machine

 ```
 sudo dhclient eth1
```

Microk8s allows to enable some addons that make kubernetes configurution easier. Some of the requireds are the storage, dns and metallb. The last will allow the system to have a load-balancer that will work as ingress to kubernetes cluster. A range of ip addresses must be given; it should be in the range of the host-only network, in order to have connection from osm node. Helm3 is also enabled to deploy microservices with charts that give facilities to automatizate some processes.

```
microk8s enable storage dns metallb helm3
```
Finally, to have a multi-node cluster, next command must be called from master node shell to obtain a unique token to be executed from other workers nodes 
```
microk8s add-node
```
Once the tokens are given , they must be provided in the workers' shells to join the master node. If there are more than three nodes, high-availability will be enabled, with at least three nodes with master status. This is actually helpull, since in the case one of them falls down o is destroyed, the information will be saved on the rest.

```
microk8s join <ip_address_master_node>:2500/<token_generated>
```

Last step is to connect both systems. A copy of kubernetes configuration is needed for that purpose. We obtain it from microk8s node, and through ssh will copy on osm machine:

```
microk8s config > config.yaml
scp config.yaml upm@<ip_address_osm_machine>:~/k8s-cluster.yaml
```

Now, this file is available in osm machine. This file must be edited; the ip address of the server must be changed from 10.0.2.15 (nat ip) to ip address of the host-only interface. Osm has the option to deploy a cluster, and with the instantation of network functions and helm charts, modules are deployed on that cluster. It can be deployed in the same kubernetes where osm is hosted or, in order to facilitate the understading of the funcionality of the system, it can be deployed on another node as well.

In adition, osm needs a VIM to have this cluster deployed. As traditional VIM such as Openstack are not available in this guide, there is another possibility: installing a dummy VIM, that points to the own machine to have all the CNFs deployed on it. Next command creates this dummy vim:

 ```
  osm vim-create --name dummy_vim --user u --password p --tenant p --account_type dummy --auth_url http://localhost/dummy
```

Next step is creating the cluster connection for osm:

```
osm k8scluster-add --creds k8s-cluster.yaml --version 1.21 --vim dummy_vim --description "External k8s cluster" --namespace osm-namespace --k8s-nets '{"net1": "osm-ext"}' microk8s-cluster
```


Once the installation is finished it is time to provide an example. This one is based on one provided in an osm hackfest, whose the purpose was creating an ldap server scheme. First of all,  we must clone directory of github where descriptors are allocated:

```
git clone --recurse-submodules -j8 https://osm.etsi.org/gitlab/vnf-onboarding/osm-packages.git
```

Once these descriptors are downloaded, it is time to create the virtual network functions.

```
cd osm-packages
```
```
osm package-validate --no-recursive openldap_knf
osm package-validate --no-recursive openldap_ns
```
```
osm package-build openldap_knf
osm package-build openldap_ns
```

```
osm nfpkg-create openldap_knf.tar.gz
osm nspkg-create openldap_ns.tar.gz
```


A script of configuration, openldad-params.yaml, must be created with thte following parameters:

```
vld:
- name: mgmtnet
  vim-network-name: osm-ext
additionalParamsForVnf:
- member-vnf-index: openldap
  additionalParamsForKdu:
  - kdu_name: ldap
    additionalParams:
      # replicaCount: 2  
      service:
        type: LoadBalancer
        loadBalancerIP: '192.168.56.51' # MetalLB IP Address
      adminPassword: osm4u
      configPassword: osm4u
      env:
        LDAP_ORGANISATION: "Example Inc."
        LDAP_DOMAIN: "example.org"
        LDAP_BACKEND: "hdb"
        LDAP_TLS: "true"
        LDAP_TLS_ENFORCE: "false"
        LDAP_REMOVE_CONFIG_AFTER_SETUP: "true"

```


After creating this script, we must launch the network function (instantation)
```
osm ns-create --ns_name ldap \
    --nsd_name openldap_ns \
    --vim_account dummy_vim \
    --config_file $HOME/openldap-params.yaml
```

After having the instace deployed, it must be tested. The first thing to do is installing openldap utils to be able to connect with openldap server

```
sudo apt-get update
sudo apt install slapd ldap-utils
```

Once is installed, it can be tested:

```
ldapsearch -x -H ldap://192.168.56.51:389 -b dc=example,dc=org -D "cn=admin,dc=example,dc=org" -w osm4u
```

