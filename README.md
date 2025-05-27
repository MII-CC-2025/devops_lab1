# DEVOPS: Introducción a Jenkins

## Requisitos

Crea una máquina virtual e instala java y jenkins en ella.

### Instalar Java

```
alvarez@jenkins:~$ javac --version

Command 'javac' not found, but can be installed with:

sudo apt install openjdk-17-jdk-headless  # version 17.0.14+7-1~24.04, or
sudo apt install openjdk-21-jdk-headless  # version 21.0.6+7-1~24.04.1
sudo apt install default-jdk              # version 2:1.17-75
sudo apt install openjdk-11-jdk-headless  # version 11.0.26+4-1ubuntu1~24.04
sudo apt install openjdk-8-jdk-headless   # version 8u442-b06~us1-0ubuntu1~24.04
sudo apt install ecj                      # version 3.32.0+eclipse4.26-2
sudo apt install openjdk-19-jdk-headless  # version 19.0.2+7-4
sudo apt install openjdk-20-jdk-headless  # version 20.0.2+9-1
sudo apt install openjdk-22-jdk-headless  # version 22~22ea-1

Ask your administrator to install one of them.
```

```
alvarez@jenkins:~$ sudo apt update

...

alvarez@jenkins:~$ sudo apt install -y openjdk-21-jdk-headless

...

alvarez@jenkins:~$ javac --version
javac 21.0.7
```


### Instalar Jenkins

https://www.jenkins.io/doc/book/installing/linux/

```
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins

```
Habilita el servicio jenkins para que arranque al iniciar la máquina:

```
sudo systemctl enable jenkins
Synchronizing state of jenkins.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable jenkins
```

Comprueba que el servicio esté ejecutándose:

```
sudo systemctl status jenkins
jenkins.service - Jenkins Continuous Integration Server
     Loaded: loaded (/lib/systemd/system/jenkins.service; enabled; vendor preset: enabled)
     Active: active (running) since ...
```

Una vez instalado, podrás acceder a jenkins mediante la IP de la máquina virtual, puerto 8080 (recuerda que debes tener abierto el puerto 8080 en el firewall).

```
https://IP_SERVER:8080/
```
Busca la clave provisional de instalación en:

```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
be1f2b1a5f78b6efde5faf8be12e7f0948d4e
```

- Instala los plugins sugeridos.

- Introduce los datos (username, password, nombre, email) del usuario administrador.



## Administrar Jenkins

- Nodos
- Plugins
- Seguridad
- Credenciales
- etc...


## Proyecto 1 (Tarea / Job) Freestyle
Aspectos básicos del funcionamiento de Jenkins

- Crea un Projecto de estilo libre:
Establece el nombre, pulsa sobre crear un projecto de estilo libre y OK.

Accederemos a la pantalla de configuración del proyecto, con un montón de opciones, dependiendo de los plugins que tengamos instalados.

Por ahora, pasaremos a la sección "Build Steps", pulsaremos sobre añadir un nuevo paso y seleccionamos Ejecutar linea de comandos (shell). 
En el cuadro de texto escribiremos el comando a ejecutar y pulsaremos sobre Guardar.

```shell
echo "Hola desde jenkins!"
```

En la pantalla del proyecto, entre otras acciones, podemos ver su workspace (directorio de trabajo), ejecutarlo con "Construir ahora" o volver a 
configurarlo de nuevo en la opción "Configurar".

Si ejecutamos el proyecto, veremos en la sección "Historia de tarea" una nueva entrada a la que podemos acceder  para comprobar su resultado, 
con la opción "Console Output".

En el panel de control tendremos todos los proyectos organizados en Vistas y/o Carpetas.




## Proyecto 2 (Tarea / Job) Freestyle

### Instalar Docker

Ahora instalaremos Docker en la máquina virtual:

```
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
```

```
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```
sudo groupadd docker

sudo usermod -aG docker $USER

sudo su - $USER
```

incluye también al usuario jenkins en el grupo docker y reinicia el servicio jenkins.

```
sudo usermod -aG docker jenkins
```

```
sudo systemctl restart jenkins

```

```
docker version
Client: Docker Engine - Community
 Version:           26.1.3
 API version:       1.45
 Go version:        go1.21.10
 Git commit:        b72abbb
 Built:             Thu May 16 08:33:49 2024
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          26.1.3
  API version:      1.45 (minimum version 1.24)
  Go version:       go1.21.10
  Git commit:       8e96db1
  Built:            Thu May 16 08:33:49 2024
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.31
  GitCommit:        e377cd56a71523140ca6ae87e30244719194a521
 runc:
  Version:          1.1.12
  GitCommit:        v1.1.12-0-g51d5e94
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

### Contenerizar App

Vamos a crear una aplicación, hola mundo, en Flask, y su fichero Dockerfile para contenerizarla. (La app y el fichero Dockerfile están disponibles en este repositorio).

Para comprobar que es correcta podemos crear la imagen y ejecutarla.

```
docker build -t flask-app .
```

```
docker run -d --name app -p 8080:8080 flask-app
```

Vamos a crear un repositorio github para mantener el control de versiones de nuestra aplicación.


### Crear Proyecto 2 Freestyle 

Ahora, creamos un nuevo proyecto en Jenkins para que cada vez que subamos una nueva versión de la App a nuestro repositorio git remoto (Github), 
se ejecute la tarea, que creará la imagen y la subirá a Docker Hub.

Primero, vamos a crear las crendenciales para DockerHub: 
Panel de control - Administrar Jenkins - Credentials - Global - Add Credential - Tipo: Username with Password - Inserta Username y Password, 
en ID pon un nombre a las credenciales.

Ahora, volvemos al proyecto:

- Accedemos al Proyecto y pulsamos en Configurar

- En "Configurar el origen del código fuente": Marcamos Git y completamos los apartados: Repository URL y Branches to build. 
  Si es un repositorio público no es necesario indicar credenciales.

- En "Disparadores de ejecuciones": marcamos GitHub hook trigger for GITScm polling.
<b>NOTA</b>: Es necesario añadir un WebHook al repositorio de Github con la URL http://<jenkins_server>/github-webhook/ para las acciones push.

- En "Entorno de ejecución" marca "Use secret text(s) or file(s)", pulsa en Añadir y selecciona "Username and password (separated)", en Username Variable introducimos: DOCKER_USERNAME, en Password Variable: DOCKER_PASSWORD y seleccionamos en Credentials las credenciales de Dockerhub creadas anteriormenete.
NOTA: Requiere haber creado previamente las crendenciales para DockerHub.

- En "Build Steps", pulsamos en "Añadir nuevo paso" y seleccionamos "Ejecutar linea de comandos (shell)". En el cuadro de texto introducimos los siguientes comandos:
```
TAG=`date "+%d%m%Y-%H%M%S"`
docker build -t jluisalvarez/flask-app:$TAG .
docker image tag jluisalvarez/flask-app:$TAG jluisalvarez/flask-app:latest
docker image ls
docker login -u="${DOCKER_USERNAME}" -p="${DOCKER_PASSWORD}"
docker push jluisalvarez/flask-app:$TAG
docker push jluisalvarez/flask-app:latest
docker image rm jluisalvarez/flask-app:$TAG jluisalvarez/flask-app:latest
```
- Finalmente, pulsamos en guardar.

Ahora, cada vez que subamos una nueva versión a nuestro repositorio Github se ejecutará la tarea, creará una imagen y la súbirá a Docker Hub, etiquetada con la fecha y hora de creación, además subirá una versiçon latest.
También, podemos iniciar la tarea manualmente.


## Proyecto 3 (Tarea / Job) Pipeline Script

- Crea un Tarea de Pipeline

- En Pipeline, elige Pipeline Script e introduce el siguiente código:

```
pipeline {

    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
           }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}
```

En el enlace Pipeline Syntax puedes acceder a un generador de código que te puede ayudar a completar la sistaxis de la pipeline.

- Pulsa en Guardar

- Ejecuta la tarea y comrpueba el resultado. Navega por las diferentes opciones de la ejecución.

### Reconfigura la Pipeline

Cambia el contenido de la Pipeline para contenerizar app y subir imagen a Docker Hub, con el siguiente código:

```
pipeline {

    agent any

    environment { 
        TAG = sh (returnStdout: true, script: 'date "+%d%m%Y-%H%M%S"').trim()
    }

    stages {
        stage("Clone Git Repository") {
            steps {
                git(
                    url: "https://github.com/MII-CC-2024/devops_jenkins_lab",
                    branch: "main",
                    changelog: true,
                    poll: true
                )
            }
        }
        stage('Build') {
            steps {
                sh '''
                echo "Building..."
                docker build -t jluisalvarez/flask_app:$TAG .
                docker tag jluisalvarez/flask_app:$TAG jluisalvarez/flask_app:latest
                '''
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
           }
        }
        stage('Publish') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'credenciales_dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh '''
                        echo "Publishing..."
                        docker login -u="${USERNAME}" -p="${PASSWORD}"
                        docker push jluisalvarez/flask_app:$TAG
                    ''' 
                
                }
            }
        }
        stage('Clean') {
            steps {
                sh '''
                echo "Cleaning..."
                docker rmi jluisalvarez/flask_app:$TAG
                ''' 
                
           }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}
```

Ejecuta la tarea y comrpueba que los resultados son los esperados.


## Proyecto 4: Pipeline from SCM (y Jenkinsfile) con AWS
Vamos a crear una pipeline que permita desplegar en EKS, auqnue en este caso, solo probaremos la conexión.

En la máquina donde esté instalado Jenkins, es necesario tener instalada las herramientas necesarias, en este caso: Docker, AWS CLI, kubectl.
Además, debemos tener un cluster EKS.

Entra en Jenkins y crea 3 Secret Text (access_key, secret_key y session_token) para guardar la configuración de acceso a AWS (aws_access_key_id, 
aws_secret_access_key, aws_session_token, respectivamente).

Crea un repositorio GitHub (aquí usaremos https://github.com/MII-CC-2025/devops_lab1).
En el IDE que desees crea un fichero Jenkinsfile con el siguiente contenido:

```
pipeline {

    agent any

    stages {
        stage("Clone Git Repository") {
            steps {
                git(
                    url: "https://github.com/MII-CC-2025/devops_lab1",
                    branch: "main",
                    changelog: true,
                    poll: true
                )
            }
        }
    
        stage('Build') {
            steps {
                sh 'docker --version'
            }
        }
        stage('Test') {
            steps {
                sh 'java --version'
           }
        }
        stage('Deploy') {
            steps {
                withCredentials([
                        string(credentialsId: 'access_key', variable: 'ACCESS_KEY'),
                        string(credentialsId: 'secret_key', variable: 'SECRET_KEY'),
                        string(credentialsId: 'session_token', variable: 'ACCESS_TOKEN'),
                    ]) {
                        withEnv([
                            "AWS_ACCESS_KEY_ID=${ACCESS_KEY}",
                            "AWS_SECRET_ACCESS_KEY=${SECRET_KEY}",
                            "AWS_SESSION_TOKEN=${ACCESS_TOKEN}",
                            "KUBECONFIG=/var/lib/jenkins/.kube/config"
                        ]) {
                            sh "echo $AWS_ACCESS_KEY_ID"
                            sh 'kubectl version'
                        }
                    }
                }
        }
    }
}

```

Crea una nueva tarea en Jenkins (Proyecto 4) de tipo pipeline cuya activación sea desde un repositorio 
(recuerda que este repositorio debes configurar un WebHook a la url de jenkins),
este debe estar configurado con el WebHook. 

Selecciona "Pipeline Script fron SMC", selecciona Git, escribe la url del repositorio (si es público, no hacen falta credenciales,
en otro caso necesitará crear un token de acceso y crear las credenciales), eligue la rama
y, finalmente, la ruta al fichero Jenkinsfile.

Ahora, cada vez que hagas un push a este repositorio, se notificará a Jenkins que active esta tarea, por lo que descargará el repositorio y
ejecutará la pipeline del fichero Jenkinsfile.



## Proyecto 5: Desplegando en Kubernetes GKE

Ahora desplagaremos a Kubernetes incluyendo el código de la Pipeline, en el fichero Jenkinsfile, del repositorio.


Necesitaremos tener un cluster GKE y debemos obtener sus credenciales; además, necesitaremos una cuenta de servicio con autorización para desplegar en el cluster.

Las credenciales del cluster, podemos incluirlas en el fichero: /var/lib/jenkins/.kube/config de la máquina virtual con la instalación de Jenkins.

La cuenta de de servicio se incluirá, en jenkins, mediante: Panel de control - Administrar Jenkins - Credentials - Global - Add Credential - Tipo: Secret file - Seleccionamos el fichero json con la cuenta de servicio y elegios un nombre en ID para las credenciales (por ejemplo, gcp_credentials).

Crea un nuevo proyecto, tipo Pipeline. En la configuración, en Pipeline, elegimos "Pipeline script from SCM" y configuramos la URL del repositorio.

Incluye, en el repositorio, el fichero Jenkinsfile con el siguiente contenido:

```
pipeline {

    agent any
    
    environment { 
        TAG = sh (returnStdout: true, script: 'date "+%d%m%Y-%H%M%S"').trim()
    }

    stages {
    
        stage("Clone Git Repository") {
            steps {
                git(
                    url: "https://github.com/MII-CC-2025/devops_lab1",
                    branch: "main",
                    changelog: true,
                    poll: true
                )
            }
        }    
        stage('Build') {
            steps {
                sh '''
                echo "Building..."
                docker build -t jluisalvarez/flask_app:$TAG .
                '''
            }
        }
        stage('Publish') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh '''
                        echo "Publishing..."
                        docker login -u="${USERNAME}" -p="${PASSWORD}"
                        docker push jluisalvarez/flask_app:$TAG
                    ''' 
                
                }
            }
        }
        stage('Clean') {
            steps {
                sh '''
                echo "Cleaning..."
                docker rmi jluisalvarez/flask_app:$TAG
                ''' 
                
           }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
                withCredentials([file(credentialsId: 'gcp_credentials', variable: 'GC_KEY')]) {
                    withEnv(["KUBECONFIG=/var/lib/jenkins/.kube/config"]) {
                      sh("gcloud auth activate-service-account --key-file=${GC_KEY}")
                      sh("envsubst < k8s/manifest.yaml | kubectl apply -f -")
                    }
                }
            }
        }
    }
}
```

