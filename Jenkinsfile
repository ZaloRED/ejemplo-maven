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
                                branches: [[name: 'feature/jenkins2' ]],
                                //Acá reemplazar por su propio repositorio
                                userRemoteConfigs: [[url: 'https://github.com/extronger/ejemplo-maven.git']]])
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
        stage('Paso 2: Sonar - Análisis Estático') {
            steps {
                script {
                    sh "echo 'Análisis Estático!'"
                    withSonarQubeEnv('sonarqube') {
                        sh "echo 'Calling sonar by ID!'"
                        // Run Maven on a Unix agent to execute Sonar.
                        sh './mvnw clean verify sonar:sonar -Dsonar.projectKey=ejemplo-maven-full-stages -Dsonar.projectName=cejemplo-maven-full-stages -Dsonar.java.binaries=build'
                    }
                }
            }
        }
        stage('Paso 3: Curl Springboot maven sleep 20') {
            steps {
                script {
                     
                     sh 'newman run /home/postman_collection.json'
                }
            }
        }
        stage('Paso 4: Detener Spring Boot') {
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
        stage('Paso 5: Subir Artefacto a Nexus') {
            steps {
                script {
                    nexusPublisher nexusInstanceId: 'nexus',
                        nexusRepositoryId: 'maven-usach-ceres',
                        packages: [
                            [$class: 'MavenPackage',
                                mavenAssetList: [
                                    [classifier: '',
                                    extension: 'jar',
                                    filePath: 'build/DevOpsUsach2020-0.0.1.jar'
                                ]
                            ],
                                mavenCoordinate: [
                                    artifactId: 'DevOpsUsach2020',
                                    groupId: 'com.devopsusach2020',
                                    packaging: 'jar',
                                    version: '0.0.1'
                                ]
                            ]
                        ]
                }
            }
        }
        stage('Paso 6: Descargar Nexus') {
            steps {
                script {
                    sh ' curl -X GET -u admin:$NEXUS_PASSWORD "http://nexus:8081/repository/maven-usach-ceres/com/devopsusach2020/DevOpsUsach2020/0.0.1/DevOpsUsach2020-0.0.1.jar" -O'
                }
            }
        }
        stage('Paso 7: Levantar Artefacto Jar en server Jenkins') {
            steps {
                script {
                    sh 'nohup java -jar DevOpsUsach2020-0.0.1.jar & >/dev/null'
                }
            }
        }
        stage('Paso 8: Testear Artefacto - Dormir(Esperar 20sg) ') {
            steps {
                script {
                    sh 'newman run /home/postman_collection.json'
                }
            }
        }
        stage('Paso 9:Detener Atefacto jar en Jenkins server') {
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