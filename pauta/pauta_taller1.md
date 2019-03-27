# Workshop 1: Devops & Bussines Agility


## Requisitos de Software para el taller

Para comenzar con el taller, debemos tener previamente instalados las siguientes herramientas:
 
* **Git client** ([instalar](https://git-scm.com/downloads))
* **aws-client** ([windows](https://docs.aws.amazon.com/cli/latest/userguide/install-windows.html), [MacOSX](https://docs.aws.amazon.com/cli/latest/userguide/install-macos.html), [Linux](https://docs.aws.amazon.com/cli/latest/userguide/install-linux.html))
* **Utilitario zip** ([Instalar](https://www.nchsoftware.com/zip/index.html?kw=how%20do%20you%20unzip%20a%20file%20mac&gclid=EAIaIQobChMIh9aMyqmg4QIVlYaRCh0HDAppEAAYASAAEgIi5vD_BwE))



## Comenzando con el taller

Para comenzar, vamos a bajar desde el repositorio todos los elementos necesarios para realizarlo desde un repositorio Git. Todo el material del taller se encuentra en este repositorio, por lo que se necesita tener previamente instalado el cliente Git.

Ubicación del repositorio:

```bash
git clone https://github.com/arkhotech/aws-workshop
```

Una vez descargado el repositorio procederemos a cargar el ambiente de trabajo en la nube. El ambiente se despliega desde un script de cloudformation, por lo que primero debemos hacer el setup correspondiente para que el script se pueda ejecutar.

**La primera actividad es crear un perfil para que Cloudformation pueda crear recursos:**

El script de Cloudformation necesita de un rol y una política de acceso para poder funcionar correctamente.  Cada rol debería tener asociada una política de acceso que indica las acciones que puede ejecutar el rol sobre los servicios.

#### Crear Política

Antes de crear el rol, crearemos una politica que nos de acceso a crear recursos sobre el servicio EC2, los que implican: VPC, Networking, Secguridad y recursos como instancias EC2.

Para crear la política debemos ingresar al servicio IAM:

* Seleccionar **Policies**
* Dar clic en **Create policy**, se abrirá el editor de políticas con 2 opciones de edición: **Visual editor** y **JSON**
* para este ejemplo usaremos **JSON**

![edit policy](../img/cf/edit_policy.png)

* Borrar todo lo que dice hay en el editor, copiar la politica en formato JSON que se muestra a continuación y pegar en el editor:

```javascript
{
    "Version": "2012-10-17",
    "Statement": [
    	{
    		"Effect": "Allow",
    		"Action" : [
    			"ec2:*"
    		],
    		"Resource": "*"
    	}
    ]   
}
```
* El resultado debe verse como sigue:

![edit policy](../img/cf/policy_edited.png)

* Dar clic en **Review policy**
* Dar el nombre **CloudFormationpolicy**

![edit policy](../img/cf/policy_name.png)

* Finalmente dar clic en **Create policy** para crear la política.

#### Crear rol y asociar política:

Arhora que la politica de ejecución para cloudformation esta creada, crear un rol.

* Ir al servicio IAM y seleccionar **Create role**
* En la siguiente pantalla:
	* En **Select type of trusted entity** dejar **AWS service**
	* Seleccionar el servicio **Cloudformation** bajo **Choose the service that will use this role**. Esto permite que cloudformation asuma el rol que estamos creando.
* Dar clic en **Next Permissions** 
* En la siguiente pantalla, escribir el nombre del rol que hemos creado anteriormente (_**cloudFormationPolicy**_), en el cuadro de texto **Filter policies**
* Dar clic en el checkbox en la política _**cloudFormationPolicy**_
* Dar clic en **Tags**. No agregaremos tags en este taller. Dar clic en **Next: Review**
* En el cuadro **Role name** agregaremos un nombre:  **cloudformationRole**
* En descripción (opcional) agregaremos algún texto descriptivo
* Dar clic en **Create role**

#### Ejecutar el script de cloudformation

* Ir al servicio "Cloudformation"
* Seleccionar "Create Stack"

![new Stack](../img/cf/new_stack.png)

* Seleccionar **"Upload a template to Amazon S3"** 

![new Stack](../img/cf/upload_script.png)

* Buscar el archivo **workshop-vpc1.yaml** ubicado en el directorio "ambiente" del repositorio que hemos descargado de Git, y luego dar clic en **"Next"**
* Llenar los campos **Stack Name**. Todos los demás campos dejarlos tal como están y dar clic en **Next**
* Dentro del Item **Permissions**, seleccionar el rol CloudformationRole (que hemos creado anteriormente)

![new Stack](../img/cf/cf_role.png)

* En la siguiente pantalla (overview), seleccionar **Create**

El proceso de creación tomará unos 2 o 3 minutos. 

#### Infraestructura

Al terminar la ejecución del script de Cloudformation, este creará la infraestructura del taller, tal como se puede ver en el siguiente diagrama:

![](../img/vpc/vpc_init.png) 


# Iniciando el taller: Codedeploy

Ahora que tenemos las infraestructura procederemos a crear manualmente una instancia EC2 la cual cumplirá el rol de nuestro servidor de aplicaciones en el taller.  Este aproach será simple e irá aumentando en profundidad a lo que avanza el taller.

El objetivo de esta sección es iniciar al usuario en el despliegue automático de aplicaciones en un servidor. Para esto usaremos una instancia EC2 y Codedeploy que simulará ser nuestro servidor de aplicaciones. 
Esta instancia tendrá instalado un servicio Nginx y se desplegará una simple pagina Web de tipo "hello world" para que el usuario pueda observar algunos detalles del despliegue. Adicionalmente, a modo de introducción solo se usarán las funcionalidades mas simples de Codedeploy.

## Rol de instancia EC2

Antes de crear la instancia EC2, debemos crear un **Rol** y una **Política**. El motivo de por que se debe crear este rol es por que la instancia EC2 necesita tener acceso a S3 para interactuar con **Codedeploy**. En caso de que la instancia no tenga acceso a S3, el despliegue fallará por timeout.

### Política EC2 pra Codedeploy

Para crear la política: 

* Acceder al servicio IAM, y seleccionar Policies
* Dar clic en Create policy, se abrirá el editor de políticas con 2 opciones de edición: Visual editor y JSON
* Al igual que el rol que cremos anteriormente, usaremos JSON

![edit policy](../img/cf/edit_policy.png)

* Eliminamos el contenido, copiamos el snipet que se muestra a continuación y pegamos en el editor.

```JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:*"
            ],
            "Resource": "*"
        }
    ]
}
```
* Dar clic en **Review policy**
* En el siguiente formulario agregar el no mbre **Ec2CodeployPolicy**, agregar una descripción descripción (opcional) y dar clic en **Create policy**

Ahora debemos crear el rol y asociar la politica recién creada.

### Crear Rol para EC2

* Ir al servicio IAM y dar clic en Roles.
* Luego dar clic en **Create role**
* Bajo el item que dice **_"Choose the service that will use this role"_**, seleccionar el servicio EC2. en este caso es el servicio que va a asumir ese rol.
* Dar clic en **Next Permission**.
* En el cuadro de texto **search** escribir el nombre que se le dió a la politica (Policy) creado anteriormente: **Ec2CodeployPolicy**. Dar clic en el checkbox para seleccionar.
* Dar clic en Tags. No agregaremos Tags en esta parte del taller.
* Asignamos el nombre para el nuevo rol (**Ec2CodedeployRole**) y una descripción opcional.
* Dar clic en **Create role**


### Crear una instancia EC2

Una vez creada la politica y el rol, procederemos a crear una instancia EC2 sobre la cual vamos a desplegar nuesrta aplicación.  Para realizar esta actividad:

* Ir al servicio **EC2**.
* Crear un nueva instancia **Launch instance**.
* Seleccionar: **_Ubuntu Server 18.04 LTS (HVM), SSD Volume Type - ami-0a313d6098716f372 (64-bit x86) / ami-01ac7d9c1179d7b74_** 

![](../img/cf/instance-ec2.png)

* Seleccionaremos una instancia de tipo **t2.micro**.
* En número de instancias, establecer en 1.
* Sobre **Network**: seleccionar la VPC que hemso creado con Cloudformation. Este debe tener el nombre Workshop 

![](../img/cf/ec2-vpc.png)

> **NOTA:** _El nombre de la VPC se establece en el momento de ejecutar el script de cloudformation.

* Seleccionar una de las redes publicas (detallar)

![](../img/cf/ec2-subnets.png)

* Seleccionar **Auto-asign Public IP** con el valor **enabled**

![](../img/cf/autoassing-ip.png)

* En IAM role seleccionar el rol que creamos para esta instancia (**Ec2CodedeployRole**).

![](../img/cf/ec2-role.png)



* Dejar los siguientes campos sin cambios a excepción de **Advanced Details**. 

![](../img/cf/advanced-details.png)

* Abrir el item y en el cuadro **User data**, copiar el siguiente código

```bash
	#!/bin/bash 
    apt-get update

    apt-get -y install ruby wget apt-transport-https ca-certificates curl software-properties-common nfs-common default-jre
    apt-get -y install nginx
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

    apt-get update 

    apt-get install -y jq
    cd /home/ubuntu 
    wget https://aws-codedeploy-us-east-2.s3.amazonaws.com/latest/install 
    chmod +x ./install 
    ./install auto 
    service codedeploy-agent start
    snap install amazon-ssm-agent --classic
    systemctl start snap.amazon-ssm-agent.amazon-ssm-agent.service

```


> **NOTA:** El código que se muestra en el snipet anterior, son instrucciones que se ejecutan al momento de crear la instancia EC2 por parte de AWS, de manera que una vez que la instancia está en estado _Running_ todas las librerías y servicios ya deben estar instalados. En este caso se instalaron 2 servicios: SSM para acceso por consola AWS y el agente de Codedeploy.

* Debe quedar de la siguiente forma:

![user data](../img/cf/user-data.png)

* Add Storage, y seleccione 30 en Size (GiB). Dejar los demás campos sin cambios. Dar clic en **Next: Add Tags**.

![user data](../img/cf/storage.png)

* Agregar un Tag haciendo clic en el botón **Add Tag**.
	* En la columna Key darle el nombre "Name" 
	* En la columna Value escribir "Workshop1"

![user data](../img/cf/tags.png)

* Dar clic en **Next: Configure Security Group** Configure security Group, dar clic en **Select existing Group**y dar clic en el grupo **FrontendSecurityGroup**

![user data](../img/cf/security-group.png)

> **NOTA:** Este grupo se ha creado al momento de crear la infraestructura por parte del servicio de cloudformation.

* Finalmente dar clic en **Review and Launch**, y finalmente en **Launch**
* En el momento de preguntar por el Keypairs, Proceed without Keypairs y confirmar.

![](../img/cf/keypairs.png)

> **NOTA:** Este Keypair se usan para acceder a la máquina a través de SSH, pero para este taller la máquina se configurará para acceder a través del servicio SSM.

#### Agregar la política para que se pueda acceder por SSM

Ahora que tenemos la máquina EC2 lista para usar, necesitamos que esta pueda ser accedida por consola ssh.  Para esto usaremos el servicio _**Session Manager**_ que es parte de las herramientas administrativas del servicio [**AWS System Manager**](https://aws.amazon.com/systems-manager). El servicio Session Manager nos da la posibilidad de abrir sesión de consola sobre una instancia Linux, usando nuestro browser. 

Para que una instancia pueda ser accedida a través de este servicio debe cumplir con 2 requisitos:

1.- Tener instalado el **ssm-agent**. Recordar que se ha instalado al momento de crear la instancia.

2.- La máquina debe tener asociado el rol para poder acceder al servicio SSM.

Para lograr esto, asociaremos el rol que usa la instancia a la política que necesita para poder usar el servicio SSM:

* En el servicios IAM abrimos el item **Roles**.
* Buscar el rol que usa la instancia. En este caso **Ec2CodedeployRole**.
* Hacer clic en el botón **Attach policies**
* Buscar la politica **AmazonEC2RoleforSSM**, seleccionar el checkbox y dar clic en el botón **Attach policy**

![](../img/cf/attach-ssm.png)

### Login en la máquina EC2

Ahora probaremos el acceso a la máquina a través del servicio SSM:

* En la consola de AWS buscar el grupo **Management & Governance**.
* Seleccionar **System Manager** bajo este item.
* Una vez dentro de la consola de **System Manager**, seleccionar **Session manager**


 y seleccionar "start session"
* Selecionar la maquina con el nombre que se le dio al tag "Name"
* Seleccionar start session
* Ejecutar sudo su ubuntu
* luego cd y enter
* ejecutar en el browser la ip de la instancia. Debería despelgar un mensaje Welcome to NGINX


## Codedeploy

* Vamos a codedeploy y seleccionamos Application. La nombraremos WorkshopSite
* El tipo de plataforma será EC2/On-Premises

## Deployment Group

* Ahora vamos a crear una deployment Group
* Damos el nombre de Deployment Group Name: Se llamará Website
* Seleccionamos el Rol CodeDeployRole (Verificar los permisos y si se creo anteriormente)
* En esta ocasión no se habalra del Blue/green, solo se hara In-Place
* Ahora lo mas importante es, identificar la o el grupo de instancias se va a desplegar la aplicación.  Las instancias se identifican por un Tag. en este caso será el nombre de la instancias
* Selecionaremos el tag Name y el nombre Workshop1
* Deployment Settings: Explicar las 3 estrategias, seleccionar AllAtOnce, Tambien exolicar "Create deplymnet Configuration"
* Deshabilitar load alancer ya que no se usará en este ejemplo
* Explicar los triggers y alarms
* Explicar rollbacks y deshabilitar


## Create deployment

VAmos a crear un deployment para desplegar una app que se encuentra en S3

### Subir aplicación a S3

El despliegue ha creado un bucket llamado **"s3://workshop.{id-cuenta}.edu"** 

* Ir al directorio site1 dentro del repositorio clonado en su máquina local
* dentro del directorio que debe tener la siguiente estructura:

```
site1/
     /appspec.yaml
     /src
     /restart-server.sh
```

* Ahora dentro de este directorio, procederemos a comprimir todo su contenido con el comando .zip: 
```bash
zip -r site.zip *
```
* Abrir la consola de AWs y seleccionar el servicio S3
* Seleccionar el Bucket que se ha creado para este taller: workshop.NNNNNNNNN.edu (donde NNNNNN es el número de cuenta de acceso)
* seleccionar el botón "upload" y subir el archivo site.zip
* Una vez subido, continuaremos con el deploy

 **Nota:** en este punto se debe revisar el archivo appsepc.yaml  y explicar el hook

## Crear Deployment Group

El deployment Group es el responsable de ejecutar el deply propiamente tal.

* Ir a Codedeply y seleccionar la aplicación que ya hemos creado atenriormente:  **WorkshopSite**
* Dar clic y seleccionar el Deployment Group
* Seleccionar "Deployment Group"
* Seleccionar luego el nombre del deployment group
* Revision Type debería ser a partir de S3
* El Location escribir:  s3://workship.NNNNNNN.edu/site.zip.   **NOTA:** (Por ahora vamos a dejar todas las opciones por defecto. Estas heredan del de la configuración de deployment group al cual peretence)
* Revision Type .zip



# Apendice A


* Aproach es el uso del AWS Cli. Se debe usar en cada cuenta un access key con un rol Adhoc
* Crear la policy, seleccionar el servicio codedepply para todos, luego dar el nombre CodedeployDevopsPolicy. (acotar).

Agregar con JSON la siguiente politica:

```JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "codedeploy:*"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:RequestedRegion": "us-east-1"
                }
            }
        }
    ]
}
```

(Buenas Prácticas) [Ver](https://aws.amazon.com/blogs/security/easier-way-to-control-access-to-aws-regions-using-iam-policies/)

### Crear un grupo

* Ir a IAM y seleccoinar nuevo grupo

* Darle un nombre al grupo: DevopsGroup

* En attach policy, escribir en el filtro el nombre de la Policy creada anteriormente:  CodedeployPolicy

* DAr clic en next step y luego Create Group

Ahora con esto podemos desplegar una aplicación 

### Crear un usuario

* Ir a IAM Users
* clic en add user
* Seleccionar un usuario o crear un usuario
* Selccionar clic en GRoup
* AddUser to group
* Selecconar el grupo DevopsGroup

Ahora estamos en condiciones de poder uasar el cliente de Codedeploy

Instalar AWS cli (Desarrollar)
