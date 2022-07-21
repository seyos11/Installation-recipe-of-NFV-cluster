Una de las partes fundamentales de un proyecto de despliegue de CNFs en OSM es empaquetar el despliegue de pods en Helm Charts. Estas sirven como plantillas de automatización para el despliegue de estas. En esta receta no se hace hincapié en la generación de estas plantillas sino como han de subirse estas plantillas a OSM.

Esta receta es tan importante debido a que los despliegues de CNFs en OSM dependen exclusivamente de Helm Charts o de Bundles. Estos últimos son charms propios de Juju que se generan con python,y también conectan estos empaquetados con Kubernetes. 

Los pasos a seguir son los siguientes:
<ol>
    <ul> 1. Creación del repositorio de Github con los archivos y templates que conforman el chart. </ul>
    <ul> 2. Creación de una página para el repositorio con Github Pages.</ul>
    <ul> 3. Realizar un empaquetamiento  de los directorios con los charts necesarios.</ul>
    <ul> 4. Creación del fichero index.yaml que ha de especificar la URL de los charts dentro del repositorio, los paquetes creados anteriormente y el nombre que lo identifica de cara al despliegue en OSM</ul>
</ol>

Una vez se dispone de todo el entorno git preparado se ha de realizar el empaquetamiento de los charts. Una vez diseñados estos, se han de empaquetar en archivos comprimidos. Para ello usamos el comando helm package junto al directorio de la aplicación de Kubernetes que se quiere desplegar. En estos directorios se encuentran los distintos charts. Un ejemplo de comando sería la siguiente:

```
helm package src/cnf1
```

Una vez empaquetados todos los charts, se dispone de un archivo index.yaml que añade una serie de metadatos y se encarga de especificar los paquetes que han de seleccionarse para disponer en el repositorio. Un ejemplo de este archivo es el sigiuente donse se dispone de 2 aplicaciones/charts.

```

apiVersion: v1
entries:
  knfQoS:
  - apiVersion: v2
    appVersion: 1.16.0
    created: "2022-03-10T12:08:28.992520265+01:00"
    description: A Helm chart for Kubernetes
    digest: 820ebb14fb6aed9a5b1f871aa6bd6cd7a391799e8353cb1ff5652f278474c07d
    name: knfQoS
    type: application
    urls:
    - https://seyos11.github.io/HelmChartQoSforOSM/knf1-0.1.0.tgz
    version: 0.1.0
  knfvCPE:
  - apiVersion: v2
    appVersion: 1.16.0
    created: "2022-03-10T12:08:28.992520265+01:00"
    description: A Helm chart for Kubernetes
    digest: 820ebb14fb6aed9a5b1f871aa6bd6cd7a391799e8353cb1ff5652f278474c07d
    name: knfvCPE
    type: application
    urls:
    - https://seyos11.github.io/HelmChartQoSforOSM/knfrouter-0.1.0.tgz
    version: 0.1.0
generated: "2022-03-10T12:08:28.991280255+01:00"
```

Una vez se dispone de esto lo único que hay que hacer hacer los commits necesarios para actulizar esta información en el repositorio de github. Con todo esto realizado lo único que queda es probar a incorporar el repositorio o repositorios requeridos a OSM y realizar los despliegeus deseados. Para añadir estas repositorios se emplea la siguiente orden:

```
    osm repo-add --type helm-chart --description "Own Helm chart repo" escenarioRedResidencial https://seyos11.github.io/HelmChartQoSforOSM/

```
