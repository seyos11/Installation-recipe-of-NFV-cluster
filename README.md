### Recetas para el despliegue y configuración de una plataforma NFV conformada por OSM, Kubernetes y Openstack
En esta repositorio se pretender ofreer una serie de recetas para el despliegue de los distintos módulos que componen la plataforma NFV montada por Víctor García durante el curso 2021-2022 para el GIROS del DIT de la UPM. Además de esto, se ofrecen una serie de diagramas con el fin de aclarar las arquitecturas de red seguidas para el desarrollo de la plataforma.

Entre los sistemas desplegados se encuentran:

 * **OSM**: Orquestador NFV para el despliegue de VNFs y CNFs.
 * **Kubernetes**: VIM para el despliegue de CNFs. 
 * **Openstack**: VIM para el despliegue de VNFs tradicionales.


El primer sistema a desplegar es OSM, ya que es el principal en un proyecto de orquestación NFV y además su despliegue es el más simple. Lo único que hay que hacer es instalar OSM en un nodo disponible mediante el uso de un par de comandos que podemos encontrar en la receta del siguiente repositorio: 
https://github.com/seyos11/Installation-recipe-of-NFV-cluster/blob/main/OSMRecipe.md

El segundo sistema a desplegar es Kubernetes. En este caso se emplea la herramienta Kubespray para la automatización del despliegue y configuración del clúster de Kubernetes. El despliegue consta de 2 nodos: un master y un worker; lo ideal sería emplear al menos dos workers para poder realizar mejores pruebas de conectividad. Se sigue la receta enfocada en un escenario virtual creado con VNX del repositorio https://github.com/seyos11/Kubespray_bare_metal.

El último sistema que se ha de desplegar es Openstack. Este es el despliegue más conflictivo de todos debido a la cantidad de redes e interfaces físicas que han de conectarse al sistema. El despliegue consta de 2 nodos: uno controlador y de red, y otro de computación; lo ideal sería emplear al menos dos nodos de computación para poder realizar mejores pruebas de conectividad. Se sigue la receta enfocada en un escenario virtual creado con VNX del repositorio https://github.com/seyos11/Openstack_cluster_kolla.

El siguiente listado muestra las recetas encontradas en este repositorio para la instalación de los distintos sistemas que conforman las plataforma NFV.

<ol>
  <li> Receta para la configuración y despliegue del clúster de Kubernetes con Kubespray: https://github.com/seyos11/Installation-recipe-of-NFV-cluster/blob/main/KubesprayRecipe.md </li>
  <li> Receta para la configuración y despliegue del clúster de Openstack con Kolla-Ansible: https://github.com/seyos11/Installation-recipe-of-NFV-cluster/blob/main/OpenstackRecipe.md </li>
  <li> Receta para la instalación de OSM: https://github.com/seyos11/Installation-recipe-of-NFV-cluster/blob/main/OSMRecipe.md</li>
</ol>

Se da otra receta de como configurar un repositorio de Helm Charts en: https://github.com/seyos11/Installation-recipe-of-OSM-and-microk8s-cluster/blob/main/HelmChartUploadRecipe.md. 

Es importante comprender como quedan las conexiones de esta plataforma a nivel físico. Para ello se muestran una serie de ilustraciones que definen dicha relación a nivel de red. Estas ilustraciones muestran las conexiones de las interfaces de los distintos dispositivos al switch físico y al router instalados en el departamento. También se muestra la nomenclatura de las interfaces empleadas.

Conexiones del nodo controlador de Openstack.

![Alt text](./images/OpenstackControllerSwitch.png?raw=true "Openstack Controller switch")

Conexiones del nodo de computación de Openstack.

![Alt text](./images/OpenstackComputeSwitch.png?raw=true "Openstack Compute switch")

Conexiones del nodo que sustenta el orquestador de OSM.

![Alt text](./images/OsmSwitch.png?raw=true "Osm connection to switch")

Conexiones del nodo maestro de Kubernetes al router.

![Alt text](./images/K8sMRouter.png?raw=true "Kubernetes Master connection to router")

Conexiones del nodo worker de Kubernetes al switch.

![Alt text](./images/K8sW1Switch.png?raw=true "Kubernetes Worker connection to switch")

Conexiones del nodo cliente.

![Alt text](./images/clienteSwitch.png?raw=true "Connection from host to switch")

Es fundamental entender las conexiones entre los distintos nodos. Se han configurado varias redes por clúster para aumentar las opciones de estos. En el futuro, se puede realizar una fase de optimización en cuanto al uso de estas redes. Un ejemplo sería seleccionar la red de gestión para la conectividad entre los nodos de K8s en lugar de usar la red del departamento. 

En las siguientes figuras se muestran una serie de diagramas que explican las arquitecturas de red seguida para las plataformas de Openstack y Kubernetes integradas con OSM. Además, se dan unas tablas con las direcciones IP de cada interfaz de todos los equipos empleados. 


Arquitectura de red para el clúster de Openstack junto a OSM.

![Alt text](./images/Openstack.png?raw=true "Connection from host to switch")

Arquitectura de red para el clúster de Kubernetes junto a OSM.

![Alt text](./images/Kubespray.png?raw=true "Connection from host to switch")

En cuanto a la configuración del switch, se ha dejado vacío este apartado. Principalmente se ha empleado para el etiquetado VLAN necesario para el escenario concreto desplegado en la simulación de la plataforma. En caso de querer realizar un escenario similar sería necesario aplicar una serie de cambios en esta configuración.

A continuación se muestran las tablas que emparejan las distintas interfaces de los equipos con las direcciones IP seleccionadas.

| Device     | Interface      | IP address  |
| :---: |   :---:       | :---: |
| `pagoda1`        | eno1         | `138.4.7.139`   |
| `pagoda2`         | eth1         | `10.0.0.32`   |
| `pagoda2`         | eth2         | `10.0.1.32`   |
| `pagoda2`         | eth3         | `---`   |
| `pagoda2`         | eth4         | `138.4.7.140`   |
| `pagoda3`         | eth1         | `138.4.7.141`   |
| `pagoda3`         | eth2         | `10.0.0.31`   |
| `pagoda3`         | eth3         | `---`   |
| `pagoda3`         | eth4         | `---`   |
| `pagoda4`         | eno1         | `138.4.7.142`   |
| `pagoda4`         | eth2         | `10.0.0.41`   |
| `pagoda5`         | eth1         | `10.0.0.11`   |
| `pagoda5`         | eth2         | `10.0.10.11`   |
| `pagoda5`         | eth3         | `---`   |
| `pagoda5`         | eth4         | `138.4.7.148`   |
| `giros9`         | enp2s0         | `138.4.7.169`   |
| `giros9`         | eth2         | `10.0.0.61`   |


En cuanto al direccionamiento de estas interfaces mencionar que es importante de cara al uso de las redes del rango de la red del departamento **138.4.7.128/25**, ya que las direcciones son limitadas. En cuanto al resto, simplemente se pueden seleccionar otras direcciones siempre que pertenezcan a la red correspondiente. 

En cuanto a la configuración de las interfaces pasa algo similar. Este nombramiento puede configurarse a gusto del administrador. En este proyecto se han usado multitud de adaptadores usb-eth para poder proporcionar a los equipos de múltiples interfaces ethernet. Las interfaces reciben su nombre gracias a reglas de configuración provistas en un script dentro del directorio **/etc/udev/rules.d** que mapea las direcciones map a nombres específicos de interfaz. 



