Esta receta es la seguida para la instalación de OSM. En primer lugar, aclarar que esta guía esta pensada para la instalación de la versión 11 de OSM.

La instalación como tal es bastante simple, solo hay que ejecutar una serie de comandos:

```
wget https://osm-download.etsi.org/ftp/osm-11.0-eleven/install_osm.sh
chmod +x install_osm.sh
./install_osm.sh 2>&1 | tee osm_install_log.txt
```
En el caso de querer instalar osm con el monitor de K8s para favorecer a posteriori el uso de herramientas de visualización como GRafana es necesario pasar al comando de instalación el flag --k8s_monitor. En nuestro caso no se ha implementado dicha instalación ya que daba un conflicto con Kubernetes y los puertos.

Una vez instalado, solo hay que seguir dos pasos: la incorporación de un VIM y, si fuera objetivo del usuario, de un cluster de Kubernetes.

Para la integración de OSM con un VIM se barajan varias posibilidades:

La primera es el uso de un dummy ficticio. Esta opción cobra sentido se se va a optar por el despliegue de CNFs en un cluster de K8s. De hecho, para el uso de un cluster de K8s este tipo de VIM es el ideal ya que sobre un clúster de Openstack, este último no tendría capacidad e gestionar ni visualizar los pdos desplegados en Kubernetes ni las redes generadas por K8s.

Este dummy ficticio es muy fácil de crear; el siguiente ocmando puede servir de insipiración:

´´´
osm vim-create --name vim_ficticio --user u --password p --tenant p --account_type dummy --auth_url http://localhost/dummy

´´´

Otra alternativa es el uso de un VIM de Openstack. En este caso hay que determinar la dirección de la API del controlador de Openstack donde este último escucha las peticiones de conexión. Hay que añadir además usuario y contraseña.

´´´
    osm vim-create --name vim_openstack --user u --password p --tenant p --account_type dummy  --account_type openstack --admin_username <usuario admin Openstack> --admin_password <contraseñaOpenstack> --auth_url  https://<direccionIP_API_Openstack>/v3
´´´

Otra opción, no comprobada, es la incorporación de un VIM en la nube a través de clásicas plataformas en la nube como Amazon Web Services o Google Cloud. 


Finalmente, hay que añadir un VIM para el despliegue de CNFs. En este caso se usa un clúster de Kubernetes; este clúster hay que añadirlo a OSM mediante el siguiente comando:


```
osm k8scluster-add --creds clusters/kubeconfig-cluster.yaml --version '1.21' --vim <VIM_NAME|VIM_ID> --description "My K8s cluster" --k8s-nets '{"net1": "osm-ext"}' cluster
```

Una vez conectado K8s a OSM ya se dispone de dos VIM, uno para el despliegue de VNFs tradicionales y otro para el despliegue de CNFs.

En el caso de querer disponer de una paralelización de usuarios se puede creer una serie de usuarios y unos proyectos asociados. Esto se puede hacer intuitivamente a través de la interfaz web de OSM. 


