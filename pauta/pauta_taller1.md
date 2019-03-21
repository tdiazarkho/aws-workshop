

# Workshop 1: Devops & Bussines Agility

## Preinstalación

Para comenzar a relizar este taller y los demás dentro de la jornada, se deben bajar los recursos para el taller, que se encuentran en un repositorio Git. Se debe tener el cliente Git (cualquiera disponible) instalado en cada una de las máquinas.

Bajar el código desde la siguiente localización:

```bash
git clone https://github.com/arkhotech/aws-workshop
```

Ahora que tenemos el código vamos a isntalar nuestro ambiente de trabajo a través de Cloudformation.  Detalles de Cloudformation, se verán mas adelante en el taller.

#### Crear una politica y un rol para que CF pueda instalar los recursos

Crear un perfil para desplegar el ambiente en el cual vamos a trabajar

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

#### Crear esta política y luego crear un rol:

* Ir a IAM y seleccionar nuevo Rol
* Seleccionar el servicio que va a asumir este rol, en este caso cloudformation
* Luego seleccionar la politica que ateriormente hemos creado, asociarla al nuevo Rol
* dar clic en crear.

#### Ejecutar el script de cloudformation

* Ir al servicio "Cloudformation"
* Seleccionar "Create Stack"
* Seleccionar **"Upload a template to Amazon S3"** 
* Buscar el archivo **workshop-vpc1.yaml** y luego dar clic en **"Next"**
* Llenar los campos **Stack Name**. Todos los demás campos dejarlos tal como están y dar clic en **Next**
* Dentro del Item **Permissions** 


aloja |  lalsls
------|--------
asdfas| asdfasdf


## Crear 



## requisitos
 
* Git client
* aws-client
* utilitario zip


## Comenzando

Crear aplicación Web Java con servicios que puedan ser llamados y probados.



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

(Buenas Practicas) [Ver](https://aws.amazon.com/blogs/security/easier-way-to-control-access-to-aws-regions-using-iam-policies/)

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

## Crear instancia EC2

En primer lugar vamos a crear un rol y una politica para poder que EC2 pueda trabajar con codedeploy. primero hay que saber que EC2 al menos tiene que tener permisos para poder llegar a S3, por lo que crearemos una politica y un rol para asignarselo a la instancia EC2

### Crear politica ede permiso (o usar una existente)

* Ir a IAM y seleccionar Policy
* Vamos a usar el editor JSON. Seleccionar el Tab que corresponde a esto
* Dentro del editor copiar el siguiente Policy Document

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
Notese que no hay restricción por Región (desarrollar)

* Dar el nombre de la policy: Ec2CodeployRole
* Entrar una descripción (opcional)
* Luego crear Policy

### Crear Rol

* Ir a IAM y dar clic en Roles
* Create role
* En donde dice "Choose the service that will use this role", dar clic en EC2 (es el servicio que va a tomar ese rol)
* Next Permission
* En search agregar el nombre del Role que hemos creado anteriormente
* Seleccionar y dar clic en Next: tags
* Dar el nombre (Ec2CodedeployRole) y la descripción


### Crear una instancia EC2

ÇProcederemos a crear una instancia EC2 donde desplegar nuesrta apilcación. Esta simplemente debe tener algunos prerequisitos como lo son tomcat y Java


* Ir al servicio EC2
* Crear un nueva instancia "launch instance"
* Seleccionar: Ubuntu Server 18.04 LTS (HVM), SSD Volume Type - ami-0a313d6098716f372 (64-bit x86) / ami-01ac7d9c1179d7b74 
* Seleccionaremos una instancia de tipo small
* Seleccionar 1 en number of instances
* Network: seleccionar la VPC que se ha creado para su cuenta con nombre workshop
* Seleccionar una de las redes publicas (detallar)
* Seleccionar auto-asign Public IP en "enabled"
* En IAM role seleccionar el rol que creamos para esta instancia  (Ec2CodedeployRole)
* Dejar tal como están todos los campos a excepción de "Advanced Details". Abrir ese item
* Donde dice "User Data", copiar el siguiente código

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


* Add Storage y seleccione 30 en Size (GiB). DEjar los demás campos por defecto.
* Agregar un Tag llamado "Name" y escribir el nombre "WorkshopI1"
* Configure security Group
* Agregra el nombre:  FrontendSG
* Agregar una descripción (opcional)
* Agregar que el puerto 22 TCP se pueda ver desde cualquier parte: __0.0.0.0/0__
* Finalmente dar clic en siguiente y luego en "Launch"
* Proceed without Keypairs

#### Agregar la politica para que se pueda acceder por SSM (Agreagr)

### Login en la máquina EC2

* Buscar el grupo "Management & Governance"
* Seleccionar System Manager
* Luego Session manager y seleccionar "start session"
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


