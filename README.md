# IBM Cloud & VMware ☁️ - Veeam 🗄️
La virtualización ha sido ampliamente implementada por empresas en centros de datos en instalaciones locales para albergar prácticamente cualquier carga de trabajo y como cualquier otro servidor en su centro de datos, realizar backup y restaurar máquinas virtuales de manera confiable es fundamental para la continuidad del negocio, por ello, se plantea una solución de Backup entre un centro de datos On-premise hacia IBM Cloud, simulando entornos completamente diferentes entre sí.

Se presenta la arquitectura de referencia:
<p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Veeam-Backup/blob/main/Images/Arquitectura1.jpg"></p>

La arquitectura presentada simula un entorno de Backup entre un centro de datos On-premise y la nube de IBM, simulando el entorno empresarial mediante la infraestructura clásica contando con un vCenter server applience y un Veeam Backup server, además del servidor o VM por replicar.


<br />


## Configuración de conexión a VPN con SSL desde clientes de MotionPro.

1. Desde la consola de IBM Cloud vaya a Gestionar > Acceso (IAM), esto lo llevara a una nueva ventana, allí de click sobre el botón Usuarios.
2. Seleccione el nombre del usuario al cual desea asignarle acceso VPN con SSL.
3. En la pagina principal del usuario de click sobre el botón Infraestructura clásica y luego de click sobre Subredes VPN.
4. seleccione el campo de Habilitar acceso VPN con SSL y de click en el botón Guardar.

Luego de haber habilitado el acceso VPN con SSL en su cuenta debe actualizar la contraseña de VPN de infraestructura clásica para poder establecer la conexión mas adelante, para esto tenga en cuenta los siguientes pasos:

1. Desde la consola de IBM Cloud vaya a Gestionar > Acceso (IAM), esto lo llevara a una nueva ventana y aquí de click sobre el botón Usuarios.
2. Seleccione el nombre del usuario al cual le quiere modificar la contraseña.
3. En la pestaña de detalles de usuario vaya a la sección Contraseña de VPN y de click sobre el botón Editar para ingresar la nueva contraseña, luego de esto de click sobre el botón Aplicar.

<br />

## Creación de la clave API para infraestructura clásica ##
Antes de iniciar el despliegue de la plataforma VMware vSphere es necesario crear una clave API para infraestructura clásica, para esto tenga en cuenta los siguientes pasos:

1. En la consola de IBM Cloud, vaya a Gestionar > Acceso (IAM), esto lo llevara a una nueva ventana y aquí de click sobre el botón Claves API/API keys.
2. Esto lo llevara a la ventana de claves API, aquí cambie la vista a claves API de la infraestructura clásica y de click sobre el botón Crear una clave de infraestructura clásica infrastructure key.
3. Luego de esto copie y descargue la clave de API en un lugar seguro para poder utilizarla mas adelante.


<p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/IAM.png"></p>

<br />

## Despliegue de Baremetal con VMware Solutions vCenter server ##

Este Baremetal nos permitirá simular un entorno On-premise.
Antes de iniciar con el desplegue del vCenter Server Applience es necesario crear el Host de vSphere con recursos suficientes para dar soporte al VMware vCenter server Applience, para esto tenga en cuenta los siguientes pasos: 

1. Desde la consola de IBM Cloud diríjase al catalogo de productos, una vez aquí busque VMware.
2. En esta nueva ventana de click sobre VMware Solutions Dedicated, esto lo llevara a la ventana de configuración del servicio de VMware Solutions Dedicated, aquí debe ingresar la siguiente información:

<p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/DespliegueBaremetal1.png"></p>

- Solutions types: Selecciones VMware vSphere, luego seleccione la casilla Create new o verifique que ya se encuentra marcada.
- Cluster configurations: New cluster.
- Cluster name: Ingrese un nombre distintivo para el cluster.
  
3. Licensing:

  Seleccione la versión de la licencia que desee teniendo en cuenta que no puede ser inferior a la versión más reciente de la imagen ISO del vCSA en este caso seleccione vSphere 7.0u2, luego de esto seleccione las casillas de Include license with purchase y VMware vCenter Server > Include license with purchase.
  
  <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/DespliegueBaremetal2.png"></p>
  
4. Bare metal server: 

- Location: Seleccione la ubicación en la cual quiere que se despliegue el servicio, en este caso se utilizó Dallas 9.
- Pod: Seleccione un pod disponible.
- CPU generation: Seleccione la casilla Cascade Lake. 
- CPU model: Seleccione la configuración que desee.
- RAM: Seleccione la cantidad de memoria necesaria.
- Number of bare metal server: Seleccione la cantidad de servidores bare metal que desee desplegar.

<p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/DespliegueBaremetal4.png"></p>
  
5. Network interface:

- Hostname prefix: Ingrese un prefijo distintivo para el hostname.
- Domain name: Ingrese un nombre distintivo para el dominio, para las instancias de vSphere el nombre de dominio debe contar de tres o más series separadas por un punto (.) con un máximo de 50 caracteres.
- Navigation type: Seleccione Public and private network.
- Uplink speed: Seleccione 10Gb.
- VLANs: Seleccione Order new VLANs.
  
6. Luego de esto dar click en el botón Create.    

Ahora al esperar el tiempo necesario de despliegue y tener el Host de vSphere con recursos suficientes se puede pasar al despliegue del vCenter. Esto se puede realizar desplegando el software de vCenter al momento de crear y configurar la VSI de infraestructura clásica de Windows en la misma VLAN del Host.

Para esto es necesario desplegar una VSI Windows de infraestructura clásica en la misma VLAN del Host creado anteriormente para poder desplegar el software de vCenter en esta máquina virtual, para esto tenga en cuenta los siguientes pasos:

  1. Desde la consola de IBM Cloud diríjase al catálogo de productos, una vez aquí busque Virtual Server for Classic.
  2. Esto llevará a una ventaja de configuración, aquí ingrese la siguiente información:

   - Type of virtual server: Seleccione Public.
   - Hostname: Ingrese un nombre distintivo. 
   - Quantity: Seleccione 1.
   - Billing method: Seleccione Monthly.

  <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/VSFC1.png"></p>

   - Location: Selecione la ubicación en la cual desee deplegar la VSI, en este caso se utilizó Dallas 9.
   - Profile: Seleccione la configuración de perfil que desee, en este caso se utilizó B1.4X16.
   -  Operating system:
   -  Vendor: Seleccione Microsoft.
   -  Version: Seleccione Windows Server 2016 Standard Edition (64 bits).
   - Attached storage disks: De click en el botón Add new y seleccione un disco adicional de 100GB.
   - Network interface
   - Uplink port speeds: Seleccione 1 Gbps non rate-limited private network uplinks.
   - Private VLAN: Seleccione la misma VLAN en la que está ubicado el Host. Para ver esta información diríjase a la lista de recursos y en la pestaña de Devices encontrará el Host creado anteriormente, de click sobre el nombre, esto lo llevará a la pestaña overview, en esta pestaña en la sección de Netowrk details encontrará las VLANs asociadas a cada interfaz, tanto la pública como la privada.

  <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/VSFC2.png"></p>
  
  Luego de completar esta información de click sobre el botón crear.

<br />

## Creación de red privada portable ##

Luego de esto es necesario crear una subred portable sobre la VLAN privada del Host la cual usaremos después, para esto tenga en cuenta los siguientes pasos:

Desde la lista de recursos en la sección Devicesingrese al servidor de vShpere, una vez aquí diríjase a la sección Network details y de click sobre la IP privada del servidor, esto le permitirá ver todas las subredes disponibles de click sobre All subnets, esto lo llevara a una nueva ventana, aquí de click sobre Create subnet.
En la pestaña de configuración ingrese la siguiente información:
 - Scope: Seleccione la misma ubicación en la que se desplego el Host, en este caso es Dallas 9.
 - Network: Seleccione Private.
 - Version: Seleccione Ipv4.
 - Size: Seleccion /29.
 - Routing: Seleccione Portable.
 - Select VLAN: Seleccione la VLAN privada del Host.
 - De click en el botón Create.
 - Una vez creada guarde la IP de la subred para usarla mas adelante

<br />

## Despliegue del vCenter Server Applience ##

Una vez desplegada la VSI Windows de infraestructura clásica se debe descargar la imagen ISO de vCenter Server Appliance. Para esto primero ejecute la conexión VPN  con el perfil de la ubicación en la cual se encuentran tanto el Host como la VSI desplegados (en este caso es Dallas 9), usando el 
<a href="https://www.ibm.com/cloud/vpn-access"> acceso VPN en IBM Cloud</a>
 luego de esto ingrese mediante Remote Desktop a la VSI desplegada anteriormente con la IP (la cual la encuentra en la pestaña de overview), el usuario y contraseña (Los cuales encuentra en la pestaña de Passwords) 
 
 <p align="center"><img width="400" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/RDP.png"></p> 
 
Una vez aquí ingrese al siguiente <a href="http://downloads.service.softlayer.com/vmware/"> enlace de descarga</a> mediante cualquier navegador para descargar la imagen ISO del vCSA seleccionando la versión de VMware-VCSA-all-6.7.0-16046470.iso, este proceso se realiza en la VSI para ahorrar tiempo y mejorar su eficiencia.

Luego de descargar el software tenga en cuenta los siguientes pasos para desplegarlo sobre la VSI:

- Monte la ISO de vCenter Server Appliance (vCSA).
- Diríjase a la carpeta vcsa-ui-installer > win32 y ejecute el installer.exe.
- Esto abrirá una la ventana del instalador, aquí de click sobre el botón Install y tenga en cuenta los siguientes pasos para la instalación.

1. STAGE 1
 - Introduction: Lea la información y de click en Next.
 - End user license agreement: Lea y acepte los términos y condiciones y de click en next.
 - Select deployment type: seleccione Embedded Platform Services Controller para desplegar el vCenter junto con el Platform Services Controller en la misma maquina.
 - Appliance deployment target:
   - ESXi host or vCenter Server name: ingrese la IP privada del Host creado anteriormente (esta la encuentra en la pestaña overview del servidor de vSphere).
   - User name: ingrese el usuario root del Host.
   - Password: ingrese la contraseña del usuario root del Host (esta la puede encontrar en la pestaña passwords del servidor de vSphere).

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/STAGE11.png"></p> 
 
   - De click en Next y acepte la advertencia de certificado.
 - Set up appliance VM:
   - VM name: Ingrese un nombre distintivo para la maquina.
   - Set root password: Ingrese la contraseña de root que desee teniendo en cuenta las especificaciones requeridas.
   - Confirm root password: confirme la contraseña ingresada anteriormente.
De click en Next.
 - Select deployment size:
   - Deployment size: Ingrese el tamaño deseado dependiendo de su aplicación, en este caso se utilizo Small.
   - Storage size: Ingrese el tamaño de almacenamiento dependiendo de su aplicación, en este caso se dejo Default.
   - De click en Next.
 - Select datastore: Seleccione la opción install on an existing datastore accesible from the target host y de click en Next.
 - Configure network settings:
   - Network: Ingrese el nombre asignado a la subred portable creada anteriormente.
   - IP version: Ingrese la versión que asigno al momento de crear la subred, en este caso es IPv4
   - IP assignment: Static.
   - IP address: Ingrese la IP que desee asignar.
   - Subnet mask or prefix length: Ingrese el prefijo para determinar el tamaño de la subnet.
   - Default gateway: Ingrese la puerda de enlace por defecto de la subred.
   - DNS Servers: Ingrese los DNS de los servidores de IBM Cloud (10.0.80.11, 10.0.80.12)
   - De click en el boton Next.
   - 
  <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/STAGE12.png"></p> 
  
 - Ready to complete stage 1: espere a que se termine el primer stage de la instalación.

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/STAGE13.png"></p>
 
 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/STAGE14.png"></p> 
 
2. STAGE 2

- Luego de terminar la configuración del Stage 1 se abrirá de manera automática el Stage 2, en caso de que esto no suceda siga las instrucciones de la ventana emergente para acceder a la configuración del stage 2.
Una vez se encuentre en la pestaña de configuración del stage 2 tenga en cuenta los siguientes pasos.

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/STAGE21.png"></p> 

- Ingrese el usuario y contraseña que le asigno al vCenter Server Appliance anteriormente
- Introduction: Lea la información y de click en Next.
- Appliance configuration: Lea la configuración y asegúrese que todos los datos estén correctos, luego de click en Next.
- SSO Configuration:
  - Single Sign-On domain name: Ingrese un nombre distintivo para el dominio del SSO.
  - Single Sign-On password: Ingrese una contraseña teniendo en cuenta las especificaciones requeridas.
  - De click en Next.

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/STAGE22.png"></p> 

- Configure CEIP: Lea la información, si esta de acuerdo seleccione la casilla de Join the VMware's Customer Experience Improvement Program (CEIP) y de click en Next.
- Ready to Complete: Verifique la información y de click en Finish.

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/STAGE23.png"></p> 

- Espere a que termine la instalación del segundo Stage.
 
<br />

## Configuración del vCenter Server ##

### Configuración inicial ###

Una vez desplegado el vCenter Server Appliance se puede pasar a la configuración de este, para hacer esto tenga en cuenta los siguientes pasos:

1. Ingrese a la pestaña de configuración del vCenter vSphere Client de la VSI creada anteriormente mediante cualquiera de las dos opciones propuestas, para esto tenga en cuenta los siguientes pasos:

 - Ejecute la conexión VPN de IBM Cloud con el perfil de la ubicación en la cual se encuentran tanto el Host como la VSI desplegados (en este caso es Dallas 9).
 - Desde la consola de IBM Cloud diríjase a la lista de recursos, una vez aquí busque la VSI que desplego anteriormente y de click sobre esta.
 - Esto lo llevara a la pestaña de Overview, aquí copie la IP publica de la maquina y péguela en un buscador para acceder a una pestaña principal de VMware.
De click sobre el botón LAUNCH VSPHERE CLIENT (HTML 5).
 - Esto lo llevara a la pestaña de inicio de VMware vCenter Single Sign-on (En caso de que ocurra un error edite el archivo de Hosts en su computador para que este reconozca la IP publica asociada al nombre de la instancia de la VSI)
 - Ingrese el usuario y la contraseña del vCenter, estos los encuentra en la pestaña Passwords.

2. Cree un Datacenter, para esto tenga en cuenta los siguientes pasos:

 - De click izquierdo sobre el nombre de la VSI, esto abrirá un menú emergente, aquí de click sobre New Datacenter.
 - Ingrese un nombre distintivo para el Datacenter y de click en Next.

3. Cree un CLuster, para esto tenga en cuenta los siguientes pasos:

 - De click izquierdo sobre el nombre del Datacenter creado anteriormente, luego de esto de click sobre New Cluster.
 - Ingrese un nombre distintivo para el cluster y de click en OK.








4. Agregue un Host, para esto tenga en cuenta los siguientes pasos:
 - De click izquierdo sobre el nombre del Datacenter creado anteriormente, luego de esto de click sobre Add Host.
 - Name and Location:
   - Host name or IP adress: Copie y pegue la IP privada del Host de vSphere creado anteriormente.
 - Connection settings
   - User name: Ingrese el usuario root del Host de vSphere.
   - Password: Ingrese la contraseña del usuario root del Host de vSphere.
 - Host summary: Lea la informacion y de click en Next.
 - Assign license: Lea la informacion y de click en Next.
 - Lockdown mode: Lea la informacion, seleccionde Disabled y de click en Next.
 - Ready to complete: Lea la informacion y de click en Finish.




### Configuración según los requerimientos ###

### Creación de un switch virtual estándar adicional ###

Antes de crear el switch adicional es necesario revisar que adaptadores están conectados al switch existente, en este caso al switch vSwitch0, para hacer esto tenga en cuenta los siguientes pasos:

- De click sobre el Host desplegado anteriormente, luego de esto de click sobre el botón Configure.
- De click sobre el botón Physical adapters en el grupo Networking.
- Esto desplegara una nueva ventana en donde se pueden ver los adaptadores físicos, su configuración y a que switch están conectados si es el caso.





Luego de esto ya se puede pasar a crear el virtual switch adicional, en este caso se creará un v-switch publico para conectar en este las interfaces 1 y 3 que son las encargadas del trafico publico en la maquina, para esto tenga en cuenta los siguientes pasos:

- De click sobre el Host desplegado anteriormente, luego de esto de click sobre el botón Configure.
- De click sobre el botón Virtual switches en el grupo Networking.
- De click sobre el botón Add Networking.
- Esto abrirá una ventana de configuración, aquí ingrese la siguiente información:
  - Select connection type: Seleccione Virtual Machine Port Group for a Standard Switch.
  - Select target device: Seleccione New Standard switch.
  - Create a Standard Switch: de click en el botón + para asignar los adaptadores deseados en el nuevo switch, luego de esto seleccione el adaptador vmnic1 y repita el proceso para el vmnic3. Si desea balancear cargas en los adaptadores utilice las flechas azules de la parte superior para asignar un adaptador a la categoría de Standby adapters.
  - Connection settings:
   - Network label: Ingrese un nombre distintivo.
   - VLAN ID: Seleccione None (0) para seleccionar la VLAN primaria.
  - Ready to Complete: De click en Finish.













Luego de crear el switch adicional puede editar el switch que estaba creado anteriormente para agregar la interfaz 2, para esto tenga en cuenta los siguientes pasos:

- De click sobre el Host desplegado anteriormente, luego de esto de click sobre el botón Configure.
- De click sobre el botón Virtual switches en el grupo Networking.
- De click sobre el nombre el virtual switch que desea editar, en este caso es el vSwitch0, luego de esto de click sobre el botón Manage physical Adapters.
- Esto abrirá una nueva pestaña de configuración, aquí de click sobre el botón +para asignar un nuevo adaptador al switch, aquí de click sobre la instancia vmnic2 y ubíquela en la región de Standby adapterspara generar el balanceo de cargas.







### Adicionar un vKernel a un vSwitch para conectarse al Host mediante la IP publica sin necesidad de utilizar VPNs ###

Para esto tenga en cuenta los siguientes pasos:

1. De click sobre el Host desplegado anteriormente, luego de esto de click sobre el botón Configure.
2. De click sobre el botón Virtual switches en el grupo Networking.
3. De click sobre el nombre del vSwitch al cual desea adicionarle un vKernel, en este caso es el vSwitch1, luego de esto de click sobre el botón Add Networking.
4. Esto abrirá una ventana de configuración, aquí ingrese la siguiente información:
  - Select connection type: Seleccione VMKernel Network Adapter.
  - Select target device: Seleccione Select an existing standard switch.
  - Port properties:
     - Network label: Ingrese un nombre distintivo.
     - VLAN ID: Seleccione None (0) para seleccionar la VLAN primaria.
     - IP settings: Seleccione IPv4.
     - Enabled services: Seleccione Management.
  - IPv4 settings: Seleccione Use static IPv4 settings, pegue la dirección IP publica del Host de vSphere, ingrese una mascara de subnet y pegue la dirección IP del gateway del Host.
  - Ready to Complete: De click en Finish.





### Crear una imagen Ubuntu.vmdk ###
Antes de iniciar con la creación de la maquina virtual es necesario subir el archivo .vmdk al datastore que creamos anteriormente, para esto tenga en cuenta los siguientes pasos:

 - De click sobre la pestaña de Storage y despliegue los menús hasta encontrar el datastore creado anteriormente, de click sobre este.
 - Una vez se encuentre en la pestaña de configuración del datastore, de click sobre Files
 - Aquí de click sobre el botón de New folder e ingrese un nombre apropiado para la carpeta, en este caso puede ser Ubuntu VM.
 - De click sobre la carpeta que acabo de crear y de click sobre el botón Upload Files y seleccione el archivo .mkv que tiene descargado en su PC.

Luego de esto puede continuar con la creación de la maquina virtual, para esto tenga en cuenta los siguientes pasos:

1. De click sobre El Host que acabomos de crear.
2. Una vez se encuentre en la pestaña de configuración del Host, de click sobre VMs.
3. Aquí de click sobre el botón de Actions y seleccione New Virtual Machine. esto desplegara un nuevo menú de configuración, aquí ingrese la siguiente información:
 - Select a creation type: seleccione la opción Create a new virtual machine.
 - Select a name and folder: Ingrese un nombre distintivo para la maquina virtual y seleccione la ubicación que desee utilizar para desplegarla.
 - Select a compute resource: seleccione el Host creado anteriormente.
 - Select storage: seleccione el datastorage creado anteriormente.
 - Select compatibility: Seleccione la opción ESXi 6.7 and later.
 - Select a guest OS:
   - Guest OS Family: Seleccione Linux.
   - Guest OS Version: Seleccione Ubuntu Linux (64.bit).
 - Customize hardware:
   - CPU: seleccione la cantidad que desee.
   - Memory: Seleccione la cantidad que desee.
   - New Hard disk: Elimine el disco que se genera por defecto de click sobre el botón ADD NEW DEVICEy seleccione Existing Hard Disk, esto abrira una nueva pestaña, aquí seleccione el disco que adiciono anteriormente en el datastore.
   - Despliegue el menú de New Hard disk y cambie el valor en Virtual Device Nodepor IDE 0,IDE (0:0) New Hard Disk.
 - Ready to complete: De click en Finish.











## Update de versión del vCenter Server ##

La actualización del vCenter de la versión 6.7 a la 7.0 se realizará desde la ISO. Ingresar al instalador de vCenter y seleccionar la opción Update, luego tenemos los siguientes pasos:

1. Introduction:
 - Stage 1: Deploy vCenter Server: El primer stage consiste en desplegar un nuevo vCenter Server.

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/update1.png"></p> 

2. End user licence agreement: Aceptar los términos de la licencia.
3. Connect to source applience:
 - vCenter Server Applience: Ingrese la IP del Server Applience.
 - HTTP post: 443.

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/update2.png"></p> 

 - Seleccionar connect to source.
 Se desplegarán más opciones para completar.
 - SSO Username: Ingresar el username del vCenter seguido de un "@" y el dominio.
 - SSO Password: Ingresar la contraseña del SSO Username.
 - Applience (OS) root password: Ingrese la contraseña del vCenter.
 - ESXi host or vCenter server name: Ingresar la IP del vCenter.
 - HTTP port: 443.
 - Username: SSO username.
 - Password: Ingresar el password del usuario.
 - Aceptar el certificate warning. 

4. vCenter Server Deployment Target:
 - ESXi host or vCenter Server: Ingresar la IP del vCenter Server.
 - HTTPS port: 443.
 - Username: SSO username.
 - Password: Ingresar el password del usuario.

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/update3.png"></p> 

Dar clic en next y acepta el Certificate Warning.   

5. Select Folder:
 - Seleccionar el Datacenter creado anteriormente y dar clic en next.

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/update4.png"></p> 
 
6. Select compute resource:
 - Seleccionar la IP del recurso y dar clic en next.

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/update5.png"></p> 

7. Set up a taget vCenter Server VM:
 - VM name: Ingresa el nombre del nuevo vCenter Server. 
 - Set root password: Ingrese la contraseña nueva para el vCenter Server.

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/update6.png"></p> 
 
8. Select Deployment Size:
 - Deployment size: Seleccionar small y verificar en el cuadro inferior la cantidad de almacenamiento y CPUs seleccionada.
 - Storage size: Seleccionar large y de clic en next.

9. Select datastore:
 - Selecciona la casilla de Show only compatible datastores.
 - Selecciona la casilla de Enable Disk Mode y dar clic en next. 

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/update7.png"></p> 

10. Configure Network Settings:
 - Network: Seleccionar la red en la que se encontrará el host del vCenter, la cual desbe ser la misma que el ESXi. La red de management es la que se utiliza como buena práctica. 
 - IP version: IPv4.
 - IP assignment: Static. 
 - Temporary IP address: Selecciona una IP dentro de la subred creada anteriormente que se encuentre disponible.
 - Subnet mask or prefix length: Ingresar la máscara de subred.
 - Default Gateway: Ingresar la IP de puerta de enlace por defecto de la subred. 
 - DNS sources: Ingresar las IP de DNS de IBM Cloud separadas por comas (10.0.80.11, 10.0.80.12) finalmente dar clic en next.

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/update8.png"></p> 
 
11. Ready to complete stage 1:
 - Revisar las especificaciones seleccionadas y dar clic en finish.

Aparecerá una barra de carga del Stage 1 y posteriormente podremos entrar en el Stage 2.

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/update9.png"></p>
 
1. Introduction
En el Stage 2 se completa el proceso de update realizando una copia de los datos desde el source applience hacia el nuevo vCenter Server desplegado. Dar clic en next.

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/update10.png"></p>
 
2. Connect to source vCenter Server:
 - Se realizará un pre-upgrate y se mostrarán resultados con algunos warnings, seleccionar cerrar.
Puedes revisar en el siguiente enlace algunos <a href="https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.update_manager.doc/GUID-8ECDD0CC-8426-44F9-A283-301F957D88A2.html"> casos comunes de advertencias y posibles soluciones</a> que te pueden indicar en el proceso de revisión. 

3. Select update data:
 - Seleccionar la data que se desea copiar: Seleccionar la primera opción de Configuration and Inventory para realizar una copia completa.

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/update11.png"></p>
 
4. Configure CEIP:
 - Seleccionar la casilla de Join the VMware's Customer Experience Improvement Program y dar clic en next.

5. Ready to complete:
 - Revisar las configuraciones finales y seleccionar next. Finalmente aceptar la advertencia sobre apagar las configuraciones de administración anteriores y dar clic en OK. Comenzará el proceso de actualización del Stage 2 y al finalizar dar clic en close. Aparecerá una ventana con un nuevo enlace a la interfaz de sesión del nuevo vCenter Server.

## Actualización del ESXi ##

Ingresar a la interfaz del ESXi colocando en el buscador la ip del host, luego ingresar al menú y luego a Lifecycle Manager en la barra izquierda.

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/update12.png"></p>

2. Deberás descargar la iso del ESXi versión 7.0 VMware-VMvisor-Installer-7.0U1c-17325551.iso de la plataforma oficial de VMware, luego seleccionar import ISO y elegir el archivo descargado.

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/update13.png"></p>

3. Crear un baseline:
 - Seleccionar Baseline, luego seleccionar create.

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/update14.png"></p>
 
 - Name and description: Ingresar un nombre para la configuración por defecto que generará el baseline.
   - Description: Agregar opcionalmente una descripción de la configuración.
   - Content: Seleccionar upgrade.

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/update15.png"></p>
 
 - Select ISO:
   - Seleccionar la ISO subida en el paso anterior y seleccionar next.

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/update16.png"></p>

 - Summary: 
   - Revisar el resumen de la configuración y dar clic en finish. 

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/update17.png"></p>
 
4. Seleccionar el menú y buscar la opción Inventory, luego seleccionar el host y buscar la opción Updates.

 <p align="center"><img width="600" src="https://github.ibm.com/jose-guerra-m/IBM-VMware-Zerto-DR/blob/main/Images/update18.png"></p>
 
5. En la opción Baselines seleccionar Attach y buscar la opción de Attact Baseline or Baselines group. 
<br />

## Creación del Zerto Virtual Manager ###

El siguiente paso será desplegar el host de Zerto dentro de IBM Cloud, para ello seguimos los siguientes pasos:

  1. En la sección de servicios busque VMware Solutions. 
  2. Seleccione VMware Solutions Shared.
  3. Complete los siguentes parámetros: 
 
   - Pricing plan: Seleccione On demand. 
   - Virtual Data Center name: Ingrese un nombre a elección para el Virtual Data Center. 
   - Deployment Topology: Seleccione Single-zone VMware virtual data center.
   - Data Center Location: Seleccione una ubicación diferente a la del vCenter desplegado.
   - vCPU limit: Ingrese un valor deseado para la capacidad del vCPU entre 1 y 512.
   - RAM limit: Ingrese un valor deseado para la memoria RAM dentre 1 y 10240 GB.
   - Recomended Services: Finalmente seleccione el servicio recomendado de Business Continuity Zerto.
<br />



## Implementación del Disaster recovery ##


## :mag: Referencias 
* <a href="https://docs.vmware.com/es/VMware-vSphere/7.0/vsphere-esxi-vcenter-server-701-configuration-guide.pdf#:~:text=Puede%20configurar%20las%20opciones%20de%20vCenter%20Server%20de,la%20forma%20de%20configuraci%C3%B3n%20preferida%20de%20vCenter%20Server."> Configuración de vCenter Server</a>.
* <a href="https://dalzssp01.vmware-solutions.cloud.ibm.com/Help/ZSSP/Content/ZSSP/CreateVPG.htm?cshid=ZSSP/CreateVPG.htm"> Configuración de VPG en Zerto </a>.
<br />

## :black_nib: Autores 
Equipo ☁️ IBM Public Cloud Customer Success.
