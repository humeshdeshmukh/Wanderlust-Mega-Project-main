pipeline {
    agent any

    environment {
        FRONTEND_DOCKER_TAG = "${params.FRONTEND_DOCKER_TAG ?: 'latest'}"
        BACKEND_DOCKER_TAG = "${params.BACKEND_DOCKER_TAG ?: 'latest'}"
        SONARQUBE_SERVER = 'SonarQube' // Name as configured in Jenkins
        DOCKER_REGISTRY = 'wanderlustacr.azurecr.io' // Update if needed
        DOCKER_CREDENTIALS_ID = 'acr-creds' // Update if needed
    }

    parameters {
        string(name: 'FRONTEND_DOCKER_TAG', defaultValue: 'latest', description: 'Docker tag for frontend')
        string(name: 'BACKEND_DOCKER_TAG', defaultValue: 'latest', description: 'Docker tag for backend')
    }

    stages {
        stage("Workspace cleanup") {
            steps {
                cleanWs()
            }
        }

        stage('Git: Code Checkout') {
            steps {
                checkout scm
            }
        }

        stage("Trivy: Filesystem scan") {
            steps {
                sh 'trivy fs --exit-code 0 --severity HIGH,CRITICAL .'
            }
        }

        stage("OWASP: Dependency check") {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    dependencyCheck additionalArguments: '--format XML', odcInstallation: 'OWASP-Dependency-Check'
                }
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        // Removed the SonarQube: Code Analysis stage as per user request

        stage('Inject env file') {
            steps {
                configFileProvider([configFile(fileId: 'frontend-env-docker', targetLocation: 'frontend/.env.docker')]) {
                    // .env.docker is now available for Docker build
                }
            }
        }

        stage("Docker: Build Images") {
            steps {
                script {
                    dir('backend') {
                        sh "docker build --platform linux/amd64 -t $DOCKER_REGISTRY/wanderlust-backend:${env.BACKEND_DOCKER_TAG} ."
                    }
                    dir('frontend') {
                        sh "docker build --platform linux/amd64 -t $DOCKER_REGISTRY/wanderlust-frontend:${env.FRONTEND_DOCKER_TAG} ."
                    }
                }
            }
        }

        stage("Docker: Push to DockerHub") {
            steps {
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login $DOCKER_REGISTRY -u $DOCKER_USER --password-stdin
                        docker push $DOCKER_REGISTRY/wanderlust-backend:${BACKEND_DOCKER_TAG}
                        docker push $DOCKER_REGISTRY/wanderlust-frontend:${FRONTEND_DOCKER_TAG}
                    '''
                }
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: '*.xml', followSymlinks: false
            build job: "Wanderlust-CD", parameters: [
                string(name: 'FRONTEND_DOCKER_TAG', value: "${params.FRONTEND_DOCKER_TAG}"),
                string(name: 'BACKEND_DOCKER_TAG', value: "${params.BACKEND_DOCKER_TAG}")
            ]
        }
    }
}
