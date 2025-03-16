pipeline {
    agent any

    parameters {
        string(name: 'ENV_NAME', defaultValue: 'dev', description: 'Environment name')
        password(name: 'MYSQL_PASSWORD', defaultValue: '', description: 'Senha do MySQL')
        string(name: 'MYSQL_PORT', defaultValue: '3306', description: 'Porta do MySQL')
    }

    environment {
        IMAGE_NAME = "custom-mysql"
        CONTAINER_NAME = "mysql-${params.ENV_NAME}"
    }

    stages {
        stage('Validação da Porta') {
            steps {
                script {
                    def port = params.MYSQL_PORT as Integer
                    if (port < 1024 || port > 65535) {
                        error "Porta inválida: ${port}. Use entre 1024 e 65535."
                    }
                }
            }
        }

        stage('Build da Imagem Docker') {
            steps {
                script {
                    writeFile file: 'Dockerfile', text: """
                    FROM mysql:latest
                    ENV MYSQL_ROOT_PASSWORD=${params.MYSQL_PASSWORD}
                    ENV MYSQL_DATABASE=DEVAPP

                    COPY init.sql /docker-entrypoint-initdb.d/
                    """
                    
                    writeFile file: 'init.sql', text: """
                    CREATE USER 'developer'@'%' IDENTIFIED BY '${params.MYSQL_PASSWORD}';
                    GRANT ALL PRIVILEGES ON *.* TO 'developer'@'%' WITH GRANT OPTION;
                    FLUSH PRIVILEGES;
                    """
                    
                    sh "docker build -t ${IMAGE_NAME} ."
                }
            }
        }

        stage('Subir Container') {
            steps {
                script {
                    // Tenta parar e remover container antigo
                    sh "docker rm -f ${CONTAINER_NAME} || true"

                    // Subir novo container
                    sh """
                    docker run -d --name ${CONTAINER_NAME} \
                    -p ${params.MYSQL_PORT}:3306 \
                    ${IMAGE_NAME}
                    """
                }
            }
        }

        stage('Verificar Container') {
            steps {
                script {
                    sh "docker ps | grep ${CONTAINER_NAME}"
                }
            }
        }
    }

    post {
        failure {
            echo "Pipeline failure. Check the logs."
        }
        success {
            echo "Container MySQL '${CONTAINER_NAME}' available on port ${params.MYSQL_PORT}"
        }
    }
}
