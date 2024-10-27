# info de la materia: ST0263-242
#
# Estudiante(s): David Grisales Posada dgrisalesp@eafit.edu.co, Juan Manuel Garzón Vargas jmgarzonv@eafit.edu.co.
#
# Profesor: Edwin Nelson Montoya, emontoya@eafit.edu.co
#

# Reto 2 Kubernetes
#
# 1. breve descripción de la actividad
#
En esta actividad desplegamos una aplicación de alta tolerancia a fallos utilizando el servicio de clúster de kubernetes de AWS.
## 1.1. Que aspectos cumplió o desarrolló de la actividad propuesta por el profesor (requerimientos funcionales y no funcionales)
- Logramos desplegar la aplicación usando el CMS Wordpress usando el clúster de kubernetes de AWS, todo esto proporcionando alta tolerancia a fallas.
- Implementamos un EFS para almacenar los datos.
- Implementamos un Load Balancer para distribuir la carga entre las instancias.
## 1.2. Que aspectos NO cumplió o desarrolló de la actividad propuesta por el profesor (requerimientos funcionales y no funcionales)
- No logramos implementar el dominio gestionado por el grupo, y consecuentemente no logramos obtener el certificado SSL del dominio.
# 2. información general de diseño de alto nivel, arquitectura, patrones, mejores prácticas utilizadas.
![image](https://github.com/user-attachments/assets/beafc664-5d40-4d5b-b9a8-2cba3bd22c00)
# 3. Descripción del ambiente de desarrollo y técnico: lenguaje de programación, librerias, paquetes, etc, con sus numeros de versiones.
  
## detalles del desarrollo.
- Usamos un clúster de Kubernetes de AWS.
- La consola de AWS CloudShell debía tener instalado kubectl y helm, se hizo por conveniencia de la información disponible.
- Usamos el sistema EFS como almacenamiento de archivos.
## detalles técnicos
- Se usaron dos nodos EC2 t2.medium para procesar las peticiones del usuario.
- Se usó un Elastic Load Balancer para distribuir la carga entre los nodos.
- Se usó un EFS para almacenar los datos.
- Se creó un security group en el EKS para permitir la conexión con el sistema de datos EFS.
  
  ![image](https://github.com/user-attachments/assets/ecde227b-2be8-44d5-9b5e-7d4c8cacf637)

## descripción y como se configura los parámetros del proyecto (ej: ip, puertos, conexión a bases de datos, variables de ambiente, parámetros, etc)
Para realizar la conexión debe cambiarse en "wordpress-deployment.yaml" la parte que dice, con los datos propios del EFS creado.
```
 csi:
    driver: efs.csi.aws.com
    volumeHandle: fs-xxxxxx::fsap-xxxxxxxx
```

# 4. Descripción del ambiente de EJECUCIÓN (en producción) lenguaje de programación, librerias, paquetes, etc, con sus numeros de versiones.
  

# IP o nombres de dominio en nube o en la máquina servidor.
Se eliminó el EKS debido a los altos costos, pero las direcciones que usaban las máquinas EC2 eran:
- 10.0.11.130 \(privada)
- 10.0.27.67  \(privada)
## descripción y como se configura los parámetros del proyecto (ej: ip, puertos, conexión a bases de datos, variables de ambiente, parámetros, etc)
1. Debe crearse el clúster de kubernetes, en nuestro caso lo llamamos eks-wp.
2. Se loggea al clúster usando CloudShell \(debe tenerse kubectl instalado) usando los comandos:
   ```
   curl -o kubectl https://amazon-eks.s3.us-east-1.amazonaws.com/1.18.9/2020-11-02/bin/linux/amd64/kubectl
   chmod +x ./kubectl
   mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
   aws eks --region us-east-1 update-kubeconfig --name eks-wp
   ```
3. Se crea una carpeta eks-wp y se accede a ella.
   ```
   mkdir eks-wp
   cd eks-wp
   ```
 4. Se copian los archivos del github presente.
 5. Se crea la instantia de EFS
 6. Se crea el security group para permitir la conexión con EFS en la VPC.
    ```
    aws ec2 create-security-group \
    --region us-east-1 \
    --group-name efs-mount-sg \
    --description "Amazon EFS for EKS, SG for mount target" \
    --vpc-id(i.e. vpc-00ab3ddf9e831f016)
    ```

    ```
    aws ec2 authorize-security-group-ingress \
    --group-id (i.e. sg-0169ed1789bf1d872) \
    --region us-east-1 \
    --protocol tcp \
    --port 2049 \
    --cidr 192.168.0.0/16
    ```
7. Se instala un driver para conectar EFS con las instancias:
   ```
    helm repo add aws-efs-csi-driver https://kubernetes-sigs.github.io/aws-efs-csi-driver/
    helm repo update
    helm upgrade --install aws-efs-csi-driver --namespace kube-system aws-efs-csi-driver/aws-efs-csi-driver
   ```
8. Se crea un access point en el EFS.
Para realizar la conexión debe cambiarse en "wordpress-deployment.yaml" la parte que dice, con los datos propios del EFS creado.
```
 csi:
    driver: efs.csi.aws.com
    volumeHandle: fs-xxxxxx::fsap-xxxxxxxx ##ID EFS :: ID EFS Access Point
```
## como se lanza el servidor.
```
kubectl apply -k .
```

## una mini guia de como un usuario utilizaría el software o la aplicación
```
kubectl get svc wordpress
```
Se copia el DNS name y se copia en el navegador, solo quedaría usar la aplicación Wordpress como se ha hecho toda la vida.
## Videosustentación
https://youtu.be/OkKj68D4UEo

# referencias:

### https://aws.amazon.com/blogs/storage/running-wordpress-on-amazon-eks-with-amazon-efs-intelligent-tiering/
### https://dev.to/aws-builders/create-a-cluster-in-amazon-eks-and-install-kubectl-4664
### https://github.com/st0263eafit/st0263-242/tree/main/eks-wp
## url de donde tomo info para desarrollar este proyecto
### https://chat.openai.com/
### https://gemini.google.com/
### https://aws.amazon.com/blogs/storage/running-wordpress-on-amazon-eks-with-amazon-efs-intelligent-tiering/
### https://dev.to/aws-builders/create-a-cluster-in-amazon-eks-and-install-kubectl-4664

