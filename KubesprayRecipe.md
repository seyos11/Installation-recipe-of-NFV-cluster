# Receta para el despliegue de kubernetes en kubespray

En primer lugar nos bajamos un zip donde se encuentran todos los ficheros que permiten el despliegue de kubespray. En él se incluyen tanto los ficheros para realizar el despliegue con ansible a la 
par que contiene un fichero xml en VNX que permite lanzar instancias virtuales en las que desplegar kubernetes. Tal caso no es el nuestro, ya que disponemos de 4 máquinas en las que realizar este despliegue.
No obstante, solo utilizaremos 3 máquinas: 1 nodo máster o de control y dos workers donde se alojarán los contenedores. La otra máquina se empleará para alojar OSM y conectarlo al clúster de Kubespray.
Esta conexión permitirá desplegar funciones de red contenerizadas.

```
wget http://idefix.dit.upm.es/download/vnx/examples/k8s/tutorial_kubespray-v01.tgz
```

```
tar -xvzf tutorial_kubespray-v01.tgz
```
Una vez extraídos todos los ficheros hay que realizar algunos ajustes en ellos. El principal archivo es el considerado inventario. En el archivo tutorial_kubespray-v01/ansible/inventory/inventory.ini hay que disponer de las siguientes líneas:


```
# ## Configure 'ip' variable to bind kubernetes services on a
# ## different ip than the default iface
# ## We should set etcd_member_name for etcd cluster. The node that is not a etcd member do not need to set the value, or can set the empty string value.
[all]
#k8s-master  ip=10.10.10.10 etcd_member_name=etcd1
pagoda1.dit.upm.es  ip=138.4.7.139 ansible_user=giros
pagoda2.dit.upm.es ip=138.4.7.140 ansible_user=giros
pagoda3.dit.upm.es ip=138.4.7.141 ansible_user=giros
r1 ip=138.4.7.129

# ## configure a bastion host if your nodes are not directly reachable
# bastion ansible_host=x.x.x.x ansible_user=some_user

[kube-master]
pagoda1.dit.upm.es

[etcd:children]
pagoda1.dit.upm.es

[kube-node]
pagoda2.dit.upm.es
pagoda3.dit.upm.es
#[calico-rr]

[k8s-cluster:children]
kube-master
kube-node
#calico-rr
```

Básicamente el cambio realizado es el de los nodos. Mientras que en el zip orginal este archivo está configurado para emplear las instancias generadas a través de VNX en este caso se configura para emplear las intancias físicas del departamento. Junto al nombre de estos hosts, se proporciona la dirección de ip de cada uno de ellos al igual que el usuario con el cual ansible accederá en estas computadoras.

Es obvio que previamente hay que configurar los hosts para que haya acceso ssh desde la máquina principal desde donde se lanza el proceso de instalación. En este caso dicha máquina es ajena a tal proceso.

Por un lado se generá claves shh en esta máquina mediante el comando *ssh-keygen*. 

Una vez generadas las claves, hacemos uso del comando *ssh-copy-id* con el cual se obtiene la clave pública generada previamente y se copia en los distintos servidores seleccionados. También se selecciona el usuario de dichos servidores con el cual se conectará a estos.

En el archivo ansible.cfg debemos cambiar la variable private_key_file con la siguiente línea:
```
private_key_file = /home/$USER/.ssh/id_rsa
```
En esta nueva variable se introduce la ruta a la clave privada generada con *ssh-keygen*


A parte de los permisos de acceso, se produce otro problema a la hora de hacer instalaciones con ansible. Normalmente, en estas instalaciones se puede preguntar por la contraseña del usuario ya que la mayoría de métodos los hace mediante el prefijo *sudo*. Para que esto 
