pipeline {
    agent any

    environment {
        NAMESPACE = 'test-wordpress'
        K8S_DIR   = '.'
    }

    stages {
        stage('Código') {
            steps {
                echo 'Fase Source: Descargando la última versión del repositorio...'
                checkout scm
            }
        }

        stage('CI') {
            steps {
                echo 'Fase CI: Validando la sintaxis de los archivos YAML...'
                sh "kubectl apply -f ${K8S_DIR}/ --dry-run=client"
            }
        }

        stage('Pruebas Continuas') {
            steps {
                echo 'Fase Pruebas: Validando conectividad con el clúster de Kubernetes...'
                sh 'kubectl cluster-info'
            }
        }

        stage('CD') {
            steps {
                echo 'Fase CD: Desplegando WordPress en el clúster de Kubernetes...'

                echo "Asegurando la existencia del namespace ${NAMESPACE}..."
                sh "kubectl create namespace ${NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -"
                
                sh "kubectl apply -f ${K8S_DIR}/PV.yml -n ${NAMESPACE}"
                sh "kubectl apply -f ${K8S_DIR}/PVC.yml -n ${NAMESPACE}"
                sh "kubectl apply -f ${K8S_DIR}/mysql-secret.yml -n ${NAMESPACE}"
                sh "kubectl apply -f ${K8S_DIR}/wordpress-configmap.yml -n ${NAMESPACE}"
                sh "kubectl apply -f ${K8S_DIR}/mysql.yml -n ${NAMESPACE}"
                sh "kubectl apply -f ${K8S_DIR}/wordpress.yml -n ${NAMESPACE}"
                
                echo 'Esperando que los pods de WordPress estén listos...'
                sh "kubectl rollout status deployment/wordpress-app -n ${NAMESPACE} --timeout=60s"
            }
        }
    }

    post {
        always {
            echo 'Mostrando el estado final de los recursos:'
            sh "kubectl get pods,svc -n ${NAMESPACE}"
        }
        success {
            echo '¡Pipeline ejecutado con éxito en las 4 etapas!'
        }
        failure {
            echo 'El pipeline falló. Por favor, revisa los logs de la etapa afectada.'
        }
    }
}
