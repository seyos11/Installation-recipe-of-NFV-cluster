# VNX Kolla-OpenStack

OpenStack scenario deployed with Kolla-ansible. The OpenStack platform is provisioned on a VNX-based virtual scenario - inspired by the [VNX Openstack Stein Lab](https://web.dit.upm.es/vnxwiki/index.php/Vnx-labo-openstack-4nodes-classic-ovs-stein).

> **IMPORTANT NOTE:**
>
> This scenario installs OpenStack **Wallaby** release.
>
> Upgrading to Xena release is expected in the future (once kolla-ansible rolls out the stable 13.00 release)

## Setup

### Pre-requisites

- Ubuntu 20.04 LTS (aka "focal")
- Python 3 (tested with Python 3.8)
- VNX ([Installation guide for Ubuntu](https://web.dit.upm.es/vnxwiki/index.php/Vnx-install-ubuntu3))

### Quick recipe (for the impacient)

```bash
git clone git@github.com:giros-dit/vnx-kolla-openstack.git
cd vnx-kolla-openstack/
python3 -m venv ansible/.kolla-venv
source ansible/.kolla-venv/bin/activate
pip install -U pip
pip install jinja2==3.0.3
pip install kolla-ansible==12.2.0
pip install 'ansible<2.10'
deactivate
sudo chown 644 conf/ssh/id_rsa
echo 'dhcp-option-force=26,1400' >> ./ansible/.kolla-venv/share/kolla-ansible/ansible/roles/neutron/templates/dnsmasq.conf.j2
sudo vnx -f openstack_lab.xml -v --create
export VNX_SCENARIO_ROOT_PATH=$(pwd)
source ansible/.kolla-venv/bin/activate
cd $VNX_SCENARIO_ROOT_PATH/ansible
kolla-ansible -i inventory/multinode --configdir kolla-config bootstrap-servers
kolla-ansible -i inventory/multinode --configdir kolla-config prechecks
kolla-ansible -i inventory/multinode --configdir kolla-config deploy
```

### Installing Kolla-Ansible (using virtual environments)

First, Kolla-ansible must be installed in the host. The recommended option is installing Kolla-ansible in a Python virtual environment. Execute the following commands to create a virtual environment within `ansible/.kolla-venv` folder, and then install kolla-ansible and its dependencies, i.e., ansible:

```bash
python3 -m venv ansible/.kolla-venv
source ansible/.kolla-venv/bin/activate
pip install -U pip
pip install jinja2==3.0.3
pip install kolla-ansible==12.2.0
pip install 'ansible<2.10'
deactivate
```

### SSH Configuration

Set proper read/write permissions for the SSH private key that Ansible will use to configure the OpenStack nodes.

```bash
sudo chown 644 conf/ssh/id_rsa
```

### Reduce virtual machines MTU

Modify dnsmasq configuration template to reduce virtual machines MTU to 1400:

```bash
echo 'dhcp-option-force=26,1400' >> ./ansible/.kolla-venv/share/kolla-ansible/ansible/roles/neutron/templates/dnsmasq.conf.j2
```

For further details on the virtual environment configuration, please visit [Kolla-ansible Virtual Environments](https://docs.openstack.org/kolla-ansible/xena/user/virtual-environments.html)

## Quickstart

### Create virtual scenario

Create a virtual scenario with VNX as follows:
```bash
sudo vnx -f openstack_lab.xml -v --create
export VNX_SCENARIO_ROOT_PATH=$(pwd)
```

### OpenStack provisioning

Activate python venv where kolla-ansible was installed:
```bash
source ansible/.kolla-venv/bin/activate
```

To provision Openstack with kolla-ansible, first prepare the target servers:
```bash
cd $VNX_SCENARIO_ROOT_PATH/ansible
kolla-ansible -i inventory/multinode --configdir kolla-config bootstrap-servers
```

Then run the `precheks` playbook to make sure that servers were properly configured with the previous playbook:
```bash
kolla-ansible -i inventory/multinode --configdir kolla-config prechecks
```

Lastly, install Openstack services in the target servers. This process will take 20 minutes roughly:
```bash
kolla-ansible -i inventory/multinode --configdir kolla-config deploy
```

Additionally, to allow external Internet access from the VMs you must configure a NAT in the host. You can easily do it using `vnx_config_nat` command distributed with VNX. Just find out the name of the public network interface of your host (i.e eth0) and execute:
```bash
sudo vnx_config_nat ExtNet eth0
```
> **WARNING:**
>
> The previous command might fail depending on your configuration of Ubuntu 20.04 LTS.
> In that case make sure `iptables` can be found under `/sbin` and execute the `vnx_config_nat` again:
 ```bash
sudo ln -s /usr/sbin/iptables /sbin/iptables
sudo vnx_config_nat ExtNet eth0
 ```

### Start demo scenario in OpenStack

Install the Python OpenStack client:
```bash
pip install python-openstackclient
```

Import credentials of Openstack admin tenant:
```bash
cd $VNX_SCENARIO_ROOT_PATH
source conf/admin-openrc.sh
```

Run the `init-runonce` utility to create demo setup - creates everything but servers.
```bash
cd $VNX_SCENARIO_ROOT_PATH
EXT_NET_CIDR='10.0.10.0/24' EXT_NET_RANGE='start=10.0.10.100,end=10.0.10.200' EXT_NET_GATEWAY='10.0.10.1' ./conf/init-runonce
```

Now you are ready to instantiate servers in the demo setup.

### Connect to Openstack Dashboard

To connect to OpenStack Dashboard, just open a web broser to http://10.0.0.11 and login with user 'admin'. The password can be obtained from conf/admin-openrc.sh script (OS_PASSWORD variable).

### Stopping the scenario

To stop the scenario preserving the configuration and the changes made:
```bash
cd $VNX_SCENARIO_ROOT_PATH
sudo vnx -f openstack_lab.xml -v --shutdown
```

## Teardown

Destroy the VNX scenario:
```bash
cd $VNX_SCENARIO_ROOT_PATH
sudo vnx -f openstack_lab.xml -v --destroy
```

To unconfigure the NAT, just execute (change eth0 by the name of your external interface):
```bash
sudo vnx_config_nat -d ExtNet eth0
```
Destroy Openstack deployment

```bash
kolla-ansible -i inventory/multinode --configdir kolla-config destroy
```

### Conflictos


Es importante realizar una modificación en el despliegue de Openstack para evitar conflictos de red. El conflicto se produce al incorporar como red de proveedor la red que es utilizada habitualmente para la conectividad con internet. En muchos proyectos va resultar esencial disponer de una red externa que habilite la comunicación con el exterior a las máquinas virtuales desplegadas. En este caso, siempre se usa la red del departamento (138.4.7.128/25) para dicha comunicación. 

El problema surge cuando en el archivo 'globals.yaml' se marca con una variable la serie de puertos/interfaces que se desea que establezcan las redes de proveedor. Entre estas interfaces se encuentra eth4, que es la que tiene el cable ethernet al router de la red del departamento. En un momento del despliegue del clúster de Openstack está interfaz se inhabilita. Esto produce que, en una serie de instrucciones a tomar posteriomente a esta inhabilitación, no se tenga acceso a internet, desencadenando una serie de fallos en el despliegue que lo interrumpen.

Al no disponer de otra conexión a Internet se toma la siguiente manipulación en la conexión de las máquinas de Openstack. En todos los nodos de Openstack se ha de configurar la ruta IP por defecto para que el camino a internet se haga a través de la interfaz de gestión. Es decir, se toma en este caso la interfaz eth1, que es la que conecta los dispositivos con la red de gestión empleada para el despleigue de ansible a través de ssh. Esta red ha de comunicarnos con otro dispositivo en al misma red (como el nodo OSM o el nodo cliente) que esté conectado a su vez a la red del departamento. Además, se ha de comprobar que dichos nodos tienen la opción de routing activada ('net.ipv4.ip_forward=1') y activar un NAT (usando el fichero creado por David de vnx_config_nat) para poder establecer dicha transformación y dar conectividad a los nodos a través de la red de gestión.

Una vez hecha esta manipulación, los nodos disponen de conectividad a internet durante el despliegue permitiendose la finalización de dicho proceso. Una vez desplegado, las instancias virtuales van a poder ser conectadas a la red de proveedor del departamento  y tener acceso externo.

Uno de los escenarios clásicos es la generación de una red externa en Openstack que disponga de un pool de direcciónes IP dentro del rango del departamento y, mediante el uso de un router y direcciones IP flotantes, se conecte a una red de autoservicio donde se desplieguen las instancias. Esto permite una capa de jerarquía y seguridad mayor, donde para acceder a las instancias se hace uso de direcciones IP flotantes mapeadas las direcciones IP de la red de autoservicio adjuntas a cada instancia virtual.

En el caso de la red del departamento es importante tener en cuenta que direcciones IP son escogidas para el pool. Al haber pocos rangos, se toma uno de tamaño mínimo (138.4.7.216-138.4.7.218) para poder realizar las pruebas necesarias.

Una cosa que sorprende es el retardo introducido al realizar conexión con la instancia a través de la IP flotante. Es algo que habría que investigar con un nivel de detalle mayor. El traceroute tampoco funciona como cabría de esperar, ya que a pesar de que el ttl mostrado en el ping es de 63 (se baja uno debido al router) el comando muestra una serie de saltos mucho mayor que 1.


En la imagen mostrada a continuación se representa la topología de red de Openstack, en las cuales se observa el escenario comentado, además de una red extra vlan.

Las redes de proveedor pueden disponer de un servicio de etiquetado VLAN. En este caso, el acceso es directo. Para ello es indispensable que las máquinas que intenten comunicarse con estas instancias virtuales estén en el mismo rango de direcciones IP, hagan uso de la misma etiqueta seleccionada para la red y esten conectados directa o indirectamnte a la misma red física. En nuestro caso se toma la interfaz eth3 para el acceso a la red de proveedor VLAN; los dispostiivos que quieran conectarse con esta red deberán tener conexión física con esta interfaz. En nuestro caso, se utiliza un switch físico que agrega todas las redes del clúster del departamento además del nodo cliente encargado de la generación de escenarios simulados. Al final, de forma indirecta, todas las instancias virtuales generadas en estos escenarios tendrán conectividad a nivel 2 con la red vlan de proveedor.


![Alt text](./topologiaRed.png?raw=true "Topología")









En el nodo controlador implementar el siguiente comando si ya se ha realizado el bootstrap-servers con error. Esto genera un perfil de libvirt que es eliminado de primeras y que es necesario tener creado para que la tarea de ansible no se interrumpa. En teoría está solucinado en la versión utilizada, ya que comprueba el perfil antes de eliminarlo, pero da error de que no existe:
De momento las pruebas de Openstack se están realizando con un nodo, lo que implica que Kubespray debe funcionar con un solo worker también. Esto es así porque se generan conflictos de docker cuando se realizan las configuraciones de kolla. 



```
sudo apparmor_parser  /etc/apparmor.d/usr.sbin.libvirtd
```

Por otro lado en docker hay que crear un fichero de configuración para que no salte un error en el servicio.  El fichero es el siguiente:/etc/systemd/system/docker.service.d/docker.conf

El contenido de dicho fichero debe ser el mostrado a continuación

```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd

```

En cuanto a la configuración hay dos ficheros que cobran especial importancia. El primero es el archivo 'globals.yaml', el cual contiene el grueso de la configuración que Kolla ha de realizar a la hora de inicializar el clúster de Openstack. En este fichero lo más importante es la selección de las interfaces que sirven en el clúster como punto de conexión a redes físicas externas, lo cual permite, acorde a la nomenclatura de Openstack, crear redes de proveedor que hacen uso de estas redes. Además, se marca la interfaz para la comunicación por túnel entre los distintos nodos que son empleadas para las redes de autoservicio. Las líneas de dicha configuración son mostradas a continuación:

```
# Opciones: ['centos', 'debian', 'rhel', 'ubuntu']
kolla_base_distro: "ubuntu"

# Opciones: [ binary, source ]
kolla_install_type: "source"


openstack_release: "wallaby"

# This should be a VIP, an unused IP on your network that will float between
# the hosts running keepalived for high-availability. If you want to run an
# All-In-One without haproxy and keepalived, you can set enable_haproxy to no
# in "OpenStack options" section, and set this value to the IP of your
# 'network_interface' as set in the Networking section below.

#Selección de VIP. Esta dirección es flotante y redirige a todos los servicios de
#Openstack. Se usa para conectarla con OSM.
kolla_internal_vip_address: "10.0.0.250"

#Selección de la interfaz de red del clúster: se elige la red de gestión
network_interface: "eth1"

#Selección de la interfaz de tunelamiento. 
tunnel_interface: "eth2"
#Selección de las interfaces para las redes externas o provider.
neutron_external_interface: "eth3,eth4"
#Nombre de los bridges virtuales creados y asociadas a las interfaces para las 
#redes externas o provider
neutron_bridge_name: "br-vlan,br-ex"
#Selección del agente de switching para neutroon. 
neutron_plugin_agent: "openvswitch"

enable_ceilometer: "yes"
enable_gnocchi: "yes"

#Habilitación de las redes de proveedor
enable_neutron_provider_networks: "yes"


```

También se selecciona una dirección ip flotante para pdoer acceder a los servicios balanceando según el puerto de acceso: **10.0.0.250**. 

El otro fichero importante es 'ml2_conf.ini'. Este fichero pertenece a la configuración propia de Openstack (no de Kolla) y se ha de tocar manualmente ya que a priori, Kolla no ofrece un fichero que abstraiga dicha configuración. Este fichero es tocado para poder determinar las interfaces físicas aptas para la generación de redes de proveedor con etiquetado VLAN. De serie, este listado viene vació, lo cual inhabilita crear redes provider con etiquetado VLAN. El script de configuración quedaría de la siguiente manera:

```
# ml2_conf.ini
[ml2]
# Changing type_drivers after bootstrap can lead to database inconsistencies
type_drivers = {{ neutron_type_drivers }}
tenant_network_types = {{ neutron_tenant_network_types }}
{% if tunnel_address_family == 'ipv6' %}
overlay_ip_version = 6
{% endif %}
{% if neutron_mechanism_drivers %}
mechanism_drivers = {{ neutron_mechanism_drivers | map(attribute='name') | join(',') }}
{% endif %}
{% if neutron_extension_drivers %}
extension_drivers = {{ neutron_extension_drivers | map(attribute='name') | join(',') }}
{% endif %}

[ml2_type_vlan]
{% if enable_ironic | bool %}
network_vlan_ranges = physnet1
{% else %}
network_vlan_ranges =
{% endif %}

[ml2_type_flat]
{% if enable_ironic | bool %}
flat_networks = *
{% else %}
flat_networks = {% for interface in neutron_external_interface.split(',') %}physnet{{ loop.index0 + 1 }}{% if not loop.last %},{% endif %}{% endfor %}
{% endif %}

[ml2_type_vxlan]
vni_ranges = 1:1000

{% if neutron_plugin_agent == 'ovn' %}
[ml2_type_geneve]
vni_ranges = 1001:2000
max_header_size = 38

[ovn]
ovn_nb_connection = {{ ovn_nb_connection }}
ovn_sb_connection = {{ ovn_sb_connection }}
ovn_metadata_enabled = True
enable_distributed_floating_ip = {{ neutron_ovn_distributed_fip | bool }}
{% endif %}
```


La clave es añadir la siguiente línea en el fichero para habilitar la 'physnet1' para el uso de VLAN: 
```
    network_vlan_ranges = physnet1
```

'physnet1' hace referencia a la interfaz eth3 configurada en el archivo 'globals.yaml'. La interfaz eth4 es la que estaría asociada a la 'physnet2'. Es decir, cada interfaz para redes externas agregada en la configuración crea, de cara a Openstack, una red física. Realmente es así ya que todas las conexiones con los nodos a parte de las redes de gestión y tunelamiento son redes físicas externas.


Los archivos de configuración pueden encontrarse en el siguiete enlace: https://github.com/seyos11/Openstack_cluster_kolla
