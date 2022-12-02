import groovy.json.JsonSlurperClassic
def jsonParse(def json) {
    new groovy.json.JsonSlurperClassic().parseText(json)
}
pipeline {
        agent any
        environment {
            NEXUS_USER         = credentials('nexus-user')
            NEXUS_PASSWORD     = credentials('nexus-password')
        }
        stages {
            stage('Paso 0: Download Code and checkout') {
            steps {
                script {
                    checkout(
                                [$class: 'GitSCM',
                                //Acá reemplazar por el nonbre de branch
                                branches: [[name: 'main' ]],
                                //Acá reemplazar por su propio repositorio
                                userRemoteConfigs: [[url: 'https://github.com/ZaloRED/ejemplo-maven.git']]])
                }
            }
            }
        stage('Paso 1: Build && Test') {
            steps {
                script {
                    sh "echo 'Build && Test!' $NEXUS_USER $NEXUS_PASSWORD"
                    sh './mvnw clean package -e'
                }
            }
        }
        stage('Paso 2: test newman maven') {
            steps {
                script {
                     
                     sh 'newman run /home/postman_collection.json'
                }
            }
        }
        stage('Paso 3: Detener Spring Boot') {
            steps {
                script {
                    sh '''
                        echo 'Process Spring Boot Java: ' $(pidof java | awk '{print $1}')
                        sleep 20
                        kill -9 $(pidof java | awk '{print $1}')
                    '''
                }
            }
        }
        stage('Paso 4: Levantar Artefacto Jar en server Jenkins') {
            steps {
                script {
                    sh 'nohup java -jar DevOpsUsach2020-0.0.1.jar & >/dev/null'
                }
            }
        }
        stage('Paso 5: Testear Artefacto con newman') {
            steps {
                script {
                    sh 'newman run /home/postman_collection.json'
                }
            }
        }
        stage('Paso 6:Detener Atefacto jar en Jenkins server') {
            steps {
                sh '''
                    echo 'Process Java .jar: ' $(pidof java | awk '{print $1}')
                    sleep 20
                    kill -9 $(pidof java | awk '{print $1}')
                '''
            }
        }
        }
}