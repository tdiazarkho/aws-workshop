Crear aplicación Web Java con servicios que puedan ser llamados y probados.



1.- Aproach es el uso del AWS Cli. Se debe usar en cada cuenta un access key con un rol Adhoc

2.- Crear la policy, seleccionar el servicio codedepply para todos, luego dar el nombre CodedeployDevopsPolicy. (acotar).

Agregar con JSON la siguiente politica:

```javascript
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

(Buenas Practicas) Ver:

https://aws.amazon.com/blogs/security/easier-way-to-control-access-to-aws-regions-using-iam-policies/

Crear un grupo

* Ir a IAM y seleccoinar nuevo grupo

* Darle un nombre al grupo: DevopsGroup

* En attach policy, escribir en el filtro el nombre de la Policy creada anteriormente:  CodedeployPolicy

* DAr clic en next step y luego Create Group

Ahora con esto podemos desplegar una aplicación 

Crear un usuario

* Ir a IAM Users
* clic en add user
* Seleccionar un usuario o crear un usuario
* Selccionar clic en GRoup
* AddUser to group
* Selecconar el grupo DevopsGroup

Ahora estamos en condiciones de poder uasar el cliente de Codedeploy

Instalar AWS cli (Desarrollar)

# Crear instancia EC2

En primer lugar vamos a crear un rol y una politica para poder que EC2 pueda trabajar con codedeploy. primero hay que saber que EC2 al menos tiene que tener permisos para poder llegar a S3, por lo que crearemos una politica y un rol para asignarselo a la instancia EC2

## Crear politica ede permiso (o usar una existente)

* Ir a IAM y seleccionar Policy
* Vamos a usar el editor JSON. Seleccionar el Tab que corresponde a esto
* Dentro del editor copiar el siguiente Policy Document

```javascript
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

## Crear Rol

* Ir a IAM y dar clic en Roles
* Create role
* En donde dice "Choose the service that will use this role", dar clic en EC2 (es el servicio que va a tomar ese rol)
* Next Permission
* En search agregar el nombre del Role que hemos creado anteriormente
* Seleccionar y dar clic en Next: tags
* Dar el nombre (Ec2CodedeployRole) y la descripción


Crear una instancia EC2

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
* Agregar que el puerto 22 TCP se pueda ver desde cualquier parte: 0.0.0.0/0
* Finalmente dar clic en siguiente y luego en "Launch"
* Proceed without Keypairs

### Agregar la politica para que se pueda acceder por SSM

## Login en la máquina EC2

* Buscar el grupo "Management & Governance"
* Seleccionar System Manager
* Luego Session manager y seleccionar "start session"
* Selecionar la maquina con el nombre que se le dio al tag "Name"
* Seleccionar start session
* Ejecutar sudo su ubuntu
* luego cd y enter
* ejecutar en el browser la ip de la instancia. Debería despelgar un mensaje Welcome to NGINX


# Codedeploy

* Vamos a codedeploy y seleccionamos Application. La nombraremos WorkshopSite
* El tipo de plataforma será EC2/On-Premises
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

