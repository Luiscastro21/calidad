pipeline {
    agent any // Esto significa que Jenkins puede usar cualquier máquina (servidor o agente) que tenga configurada para ejecutar este proceso.

    environment {
        // Aquí definimos variables que usaremos en el pipeline para que sea más fácil de leer y modificar.
        DOCKER_REGISTRY = 'docker.io' // El registro de Docker que usas (Docker Hub)
        DOCKER_REPO_USERNAME = 'luiscastro21' // ¡IMPORTANTE! Tu nombre de usuario de Docker Hub
        PROJECT_ROOT_DIR = '.' // Representa la carpeta raíz de tu proyecto en Jenkins
        BACKEND_DIR = 'backend' // Nombre de la carpeta de tu backend
        FRONTEND_DIR = 'client' // Nombre de la carpeta de tu frontend (React)
    }

    stages {
        // Cada "stage" es una fase del proceso de CI/CD
        stage('1. Obtener Código Fuente') {
            steps {
                echo 'Clonando el repositorio Git...'
                // ¡IMPORTANTE! Reemplaza 'https://github.com/tu-usuario/tu-repositorio-mern.git' con la URL real de tu repositorio Git.
                // Por ejemplo: 'https://github.com/luiscastro21/mern-crud-auth.git' (si ese es tu repo)
                git branch: 'main', url: 'https://github.com/TU_USUARIO/TU_REPOSITORIO_MERN.git'
            }
        }

        stage('2. Construir Imágenes Docker') {
            steps {
                script { // 'script' permite ejecutar comandos más complejos de Groovy o shell
                    echo 'Construyendo imagen del Backend...'
                    dir("${BACKEND_DIR}") { // Entra a la carpeta 'backend'
                        // Construye la imagen del backend, la etiqueta con el usuario de Docker Hub y el número de la build de Jenkins (ej. luiscastro21/mern-backend:1)
                        sh "docker build -t ${DOCKER_REPO_USERNAME}/mern-backend:${env.BUILD_NUMBER} -f Dockerfile.backend ."
                    }

                    echo 'Construyendo imagen del Frontend...'
                    dir("${FRONTEND_DIR}") { // Entra a la carpeta 'client' (donde está tu React)
                        // Construye la imagen del frontend
                        sh "docker build -t ${DOCKER_REPO_USERNAME}/mern-frontend:${env.BUILD_NUMBER} -f Dockerfile.frontend ."
                    }
                }
            }
        }

        stage('3. Subir Imágenes Docker a Docker Hub') {
            steps {
                script {
                    echo 'Iniciando sesión en Docker Hub...'
                    // 'withCredentials' es una forma segura de usar tus credenciales de Docker Hub que configuraste en Jenkins.
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        // Inicia sesión en Docker Hub usando las credenciales seguras.
                        sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin ${DOCKER_REGISTRY}"

                        echo 'Subiendo imagen del Backend...'
                        sh "docker push ${DOCKER_REPO_USERNAME}/mern-backend:${env.BUILD_NUMBER}"
                        echo 'Subiendo imagen del Frontend...'
                        sh "docker push ${DOCKER_REPO_USERNAME}/mern-frontend:${env.BUILD_NUMBER}"

                        echo 'Cerrando sesión de Docker Hub...'
                        sh "docker logout ${DOCKER_REGISTRY}"
                    }
                }
            }
        }

        stage('4. Desplegar en Kubernetes') {
            steps {
                script {
                    echo 'Actualizando los manifiestos de Kubernetes con el nuevo tag de imagen...'
                    // Estos comandos 'sed' modifican temporalmente tus archivos YAML de Kubernetes
                    // para que usen el número de build actual de Jenkins como el tag de la imagen.
                    // Esto fuerza a Kubernetes a descargar la nueva versión.
                    // ¡IMPORTANTE! Asegúrate de que 'luiscastro21' aquí coincide con tu DOCKER_REPO_USERNAME
                    sh "sed -i 's|luiscastro21/mern-backend:.*|${DOCKER_REPO_USERNAME}/mern-backend:${env.BUILD_NUMBER}|g' ${PROJECT_ROOT_DIR}/k8s/backend-deployment.yaml"
                    sh "sed -i 's|luiscastro21/mern-frontend:.*|${DOCKER_REPO_USERNAME}/mern-frontend:${env.BUILD_NUMBER}|g' ${PROJECT_ROOT_DIR}/k8s/frontend-deployment.yaml"

                    echo 'Aplicando los manifiestos actualizados en Kubernetes...'
                    // Este comando le dice a Kubernetes que aplique los cambios en tus archivos YAML.
                    // Kubernetes detectará el cambio en el tag de la imagen y actualizará tus aplicaciones.
                    sh "kubectl apply -f ${PROJECT_ROOT_DIR}/k8s/"
                }
            }
        }
    }
    post {
        // Acciones que se ejecutan al final del pipeline, sin importar si falla o tiene éxito
        always {
            echo 'Limpiando el workspace de Jenkins...'
            cleanWs() // Elimina todos los archivos del espacio de trabajo de Jenkins para el siguiente build
        }
        success {
            echo '¡Despliegue exitoso!'
        }
        failure {
            echo '¡El despliegue ha fallado! Revisa los logs.'
        }
    }
}