def versionPom = ""
pipeline{
    agent {
        kubernetes {
            // Rather than inline YAML, in a multibranch Pipeline you could use: yamlFile 'jenkins-pod.yaml'
            // Or, to avoid YAML:
            // containerTemplate {
            //     name 'shell'
            //     image 'ubuntu'
            //     command 'sleep'
            //     args 'infinity'
            // }
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: maven
    command:
    - sleep
    args:
    - infinity
  - name: imgkaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - /busybox/cat
    tty: true
'''
            // Can also wrap individual steps:
            // container('shell') {
            //     sh 'hostname'
            // }
            defaultContainer 'shell'
        }
    }

    stages {
        
        stage('Unit Tests') {
          steps {
            echo '''04# Stage - Unit Tests (develop y main): Lanzamiento de test unitarios.
            '''
            sh "mvn test"
            junit "target/surefire-reports/*.xml"
                }
        }

        stage('Package') {
            steps {
            echo '''07# Stage - Package
(develop y main): Generación del artefacto .jar (SNAPSHOT)
'''
                sh 'mvn package -DskipTests'
            }
        }

        stage('Build & Push') {
            steps {
            echo '''08# Stage - Build & Push
(develop y main): Construcción de la imagen con Kaniko y subida de la misma a repositorio personal en Docker Hub.
Para el etiquetado de la imagen se utilizará la versión del pom.xml
'''
                container('imgkaniko') {
                   
                    script {
                        def APP_IMAGE_NAME = "app-pf-backend"
                        def APP_IMAGE_TAG = "1.0.0" //Aqui hay que obtenerlo de POM.txt
                        withCredentials([usernamePassword(credentialsId: 'idCredencialesDockerHub', passwordVariable: 'idCredencialesDockerHub_PASS', usernameVariable: 'idCredencialesDockerHub_USER')]) {
                            AUTH = sh(script: """echo -n "${idCredencialesDockerHub_USER}:${idCredencialesDockerHub_PASS}" | base64""", returnStdout: true).trim()
                            command = """echo '{"auths": {"https://index.docker.io/v1/": {"auth": "${AUTH}"}}}' >> /kaniko/.docker/config.json"""
                            sh("""
                                set +x
                                ${command}
                                set -x
                                """)
                            sh "/kaniko/executor --dockerfile Dockerfile --context ./ --destination ${idCredencialesDockerHub_USER}/${APP_IMAGE_NAME}:${APP_IMAGE_TAG}"
                            sh "/kaniko/executor --dockerfile Dockerfile --context ./ --destination ${idCredencialesDockerHub_USER}/${APP_IMAGE_NAME}:latest --cleanup"
                        }
                    }
                } 
            }
        }
        
        /*stage('SonarQube analysis') {
          steps {
            withSonarQubeEnv(credentialsId: "sonar-token", installationName: "Sonarqube"){
                sh "mvn clean verify sonar:sonar -DskipTests"
            }
          }
        }

        stage('Quality Gate') {
          steps {
            timeout(time: 10, unit: "MINUTES") {
              script {
                def qg = waitForQualityGate(webhookSecretId: 'sonar-token')
                if (qg.status != 'OK') {
                   error "Pipeline aborted due to quality gate failure: ${qg.status}"
                }
              }
            }
          }
        } */


    }

}