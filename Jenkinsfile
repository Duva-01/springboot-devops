pipeline {
  agent any

  environment {
    // Nombre de tu repositorio en Docker Hub
    IMAGE_NAME      = "Duva-01/microservice"
    // Ruta relativa a tu Helm chart dentro del repositorio
    CHART_DIR       = "charts/microservice"
    // ID de la credencial que guardaste en Jenkins para DockerHub
    DOCKERHUB_CREDS = "dockerhub-creds"
    // Nombre de la release de Helm en Kubernetes
    HELM_RELEASE    = "micro-prod"
    // Namespace de Kubernetes donde estás desplegando
    HELM_NAMESPACE  = "default"
    // Contexto de kubeconfig para tu clúster Kind
    KUBE_CONTEXT    = "kind-devops-demo"
  }

  stages {
    stage('Checkout') {
      steps {
        // Clona el repositorio público
        git url: 'https://github.com/Duva-01/springboot-devops.git',
            branch: 'main'
      }
    }

    stage('Build JAR') {
      steps {
        // Se mete en la carpeta microservice y compila sin tests
        dir('microservice') {
          sh './mvnw clean package -DskipTests'
        }
      }
    }

    stage('Build & Push Docker Image') {
      steps {
        script {
          // Usamos el número de build como etiqueta única
          def imageTag = "${BUILD_NUMBER}"
          dir('microservice') {
            // Construye la imagen con la ruta del Dockerfile
            sh "docker build -t ${IMAGE_NAME}:${imageTag} ."
          }
          // Hace login en DockerHub usando las credenciales guardadas y luego push
          withCredentials([usernamePassword(
                             credentialsId: "${DOCKERHUB_CREDS}",
                             usernameVariable: 'DOCKER_USR',
                             passwordVariable: 'DOCKER_PW')]) {
            sh """
              echo "$DOCKER_PW" | docker login -u "$DOCKER_USR" --password-stdin
              docker push ${IMAGE_NAME}:${imageTag}
            """
          }
        }
      }
    }

    stage('Deploy with Helm') {
      steps {
        script {
          def imageTag = "${BUILD_NUMBER}"
          // Ejecuta helm upgrade/--install para actualizar la release
          sh """
            helm upgrade ${HELM_RELEASE} ${CHART_DIR} \
              --install \
              --namespace ${HELM_NAMESPACE} \
              --set image.tag=${imageTag} \
              --kube-context ${KUBE_CONTEXT}
          """
        }
      }
    }
  }

  post {
    always {
      // (Opcional) Borra la imagen local para no consumir espacio
      script {
        def imageTag = "${BUILD_NUMBER}"
        sh "docker rmi ${IMAGE_NAME}:${imageTag} || true"
      }
    }
    success {
      echo "Pipeline finalizado con éxito: versión ${BUILD_NUMBER}"
    }
    failure {
      echo "¡Error en el pipeline! Revisa los logs para más detalles."
    }
  }
}
