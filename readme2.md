En esta repositorio se pretender ofreer una serie de recetas para el despliegue de los distintos módulos que componen la plataforma NFV montada por Víctor García durante el curso 2021-2022.


Entre los sistemas desplegados se encuentran:
<ol>
  <li> OSM: Orquestador NFV para el despliegue de VNFs y CNFs.</li>
  <li> Kubernetes: VIM para el despliegue de CNFs. </li>
  <li> Openstack: VIM para el despliegue de VNFs tradicionales.</li>
</ol>

El primer sistema a desplegar es OSM, ya que es el principal en un proyecto de orquestación NFV y además su despliegue es el más simple. Lo único que hay que hacer es instalar OSM en un nodo disponible mediante el uso de un par de comandos que podemos encontrar en la receta del siguiente repositorio: 


El segundo sistema a desplegar es Kubernetes. En este caso se emplea la herramienta Kubespray para la automatización del despliegue y configuración del clúster de Kubernetes. El despliegue consta de 2 nodos: un master y un worker; lo ideal sería emplear al menos dos workers para poder realizar mejores pruebas de conectividad. Se sigue la receta del repositorio https://github.com/seyos11/Kubespray_bare_metal

El último sistema que se ha de desplegar es Openstack. Este es el despliegue más conflictivo de todos debido a la cantidad de redes e interfaces físicas que han de conectarse al sistema. El despliegue consta de 2 nodos: uno controlador y de red, y otro de computación; lo ideal sería emplear al menos dos nodos de computación para poder realizar mejores pruebas de conectividad. Se sigue la receta del repositorio https://github.com/seyos11/Openstack_cluster_kolla

Es importante comprender como quedan las conexiones de esta plataforma a nivel físico. Para ello se muestran una serie de ilustraciones que definen dicha relación a nivel de red

![Alt text](./images/OpenstackControllerSwitch.png?raw=true "Openstack Controller switch")
![Alt text](./images/OpenstackComputeSwitch.png?raw=true "Openstack Compute switch")
![Alt text](./images/OsmSwitch.png?raw=true "Osm connection to switch")
![Alt text](./images/clienteSwitch.png?raw=true "Connection from host to switch")
![Alt text](./images/K8sMRouter.png?raw=true "Kubernetes Master connection to router")
![Alt text](./images/K8sW1Switch.png?raw=true "Kubernetes Worker connection to switch")



