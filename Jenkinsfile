pipeline {
    agent any

    environment {
        NAMESPACE = 'test-wordpress'
        K8S_DIR   = '.'
    }
    stages {
        stage('CI') {
            steps {
                echo 'Etapa_2: Validando conexión al clúster y la sintaxis de los archivos YAML'
                echo 'Prueba rapida de conexión'
                sh 'kubectl cluster-info'
                echo 'Listando archivos descargados de GitHub'
                sh "ls -la ${K8S_DIR}"
                echo 'Simulación profunda en el clúster'
                sh "kubectl apply -f ${K8S_DIR}/ --dry-run=server"
            }
        }

        stage('CD') {
            steps {
                echo 'Etapa_4: Desplegando WordPress & MYSQL en el clúster de Kubernetes'

                echo "Verificando existencia del namespace ${NAMESPACE}"
                sh "kubectl create namespace ${NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -"
                
                sh "kubectl apply -f ${K8S_DIR}/PV.yml -n ${NAMESPACE}"
                sh "kubectl apply -f ${K8S_DIR}/PVC.yml -n ${NAMESPACE}"
                sh "kubectl apply -f ${K8S_DIR}/mysql-secret.yml -n ${NAMESPACE}"
                sh "kubectl apply -f ${K8S_DIR}/wordpress-configmap.yml -n ${NAMESPACE}"
                sh "kubectl apply -f ${K8S_DIR}/mysql.yml -n ${NAMESPACE}"
                sh "kubectl apply -f ${K8S_DIR}/wordpress.yml -n ${NAMESPACE}"
                
                echo 'Verificando estado de los deployment'
                sh "kubectl rollout status deployment/mysqldb -n ${NAMESPACE} --timeout=90s"
                sh "kubectl rollout status deployment/wordpress-app -n ${NAMESPACE} --timeout=90s"
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
