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
su - $USER
```

Microk8s allows to enable some addons that make kubernetes configurution easier. Some of the requireds are the storage, dns and metallb. The last will allow the system to have a load-balancer that will work as ingress to kubernetes cluster. A range of ip addresses must be given; it should be in the range of the host-only network, in order to have connection from osm node.
```
microk8s enable storage dns metallb
```
Finally, to have a multi-node cluster, next command must be called from master node shell to obtain a unique token to be executed from other workers nodes 
```
microk8s add-node
```
Once the tokens are given , they must be provided in the workers' shells to join the master node. If there are more than three nodes, high-availability will be enabled, with at least three nodes with master status. This is actually helpull, since in the case one of them falls down o is destroyed, the information will be saved on the rest.

```
microk8s join <ip_address_master_node>:2500/<token_generated>
```
