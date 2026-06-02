pipeline {
    agent any

    environment {
        NAMESPACE = 'test_wordpress'
    }

    stages {
        stage('Codigoo') {
            steps {
                checkout scm
            }
        }

        stage('Validar Conexión K8s') {
            steps {
                sh 'kubectl cluster-info'
            }
        }

        stage('Desplegar WordPress') {
            steps {
                echo 'Aplicando manifiestos de Kubernetes...'
                sh "kubectl apply -f k8s/pvc.yaml -n ${NAMESPACE}"
                sh "kubectl apply -f k8s/secret.yaml -n ${NAMESPACE}"
                sh "kubectl apply -f k8s/mysql-dep.yaml -n ${NAMESPACE}"
                sh "kubectl apply -f k8s/mysql-svc.yaml -n ${NAMESPACE}"
                sh "kubectl apply -f k8s/wp-dep.yaml -n ${NAMESPACE}"
                sh "kubectl apply -f k8s/wp-svc.yaml -n ${NAMESPACE}"
            }
        }

        stage('Verificar Estado') {
            steps {
                sh "kubectl get pods,svc -n ${NAMESPACE}"
            }
        }
    }
}
