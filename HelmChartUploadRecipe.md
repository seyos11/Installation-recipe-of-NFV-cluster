Una de las partes fundamentales de un proyecto de despliegue de CNFs en OSM es empaquetar el despliegue de pods en Helm Charts. Estas sirven como plantillas de automatización para el despliegue de estas. En esta receta no se hace hincapié en la generación de estas plantillas sino como han de subirse estas plantillas a OSM.

Esta receta es tan importante debido a que los despliegues de CNFs en OSM dependen exclusivamente de Helm Charts o de Bundles. Estos últimos son charms propios de Juju que se generan con python,y también conectan estos empaquetados con Kubernetes. 

Los pasos a seguir son los siguientes:
<ol>
    <ul>Creación del repositorio de Github con los archivos y templates que conforman el chart. </ul>
    <ul> Creación de una página para el repositorio con Github Pages.</ul>
    <ul> Realizar un empaquetamiento  de los directorios con los charts necesarios.</ul>
    <ul> Creación del fichero index.yaml que ha de especificar la URL de los charts dentro del repositorio, los paquetes creados anteriormente y el nombre que lo identifica de cara al despliegue en OSM</ul>
</ol>
