pipeline {
    agent { label 'Test_node' }

    environment {
        GIT_REPO_URL = 'https://github.com/VASANTHAPUVVADA/real-time-SCM.git'
        MAVEN_HOME = tool name: 'Maven3', type: 'ToolLocation' // Ensure Maven is installed in Jenkins
        SONARQUBE = 'SonarQube' // Set your SonarQube server in Jenkins configuration
        NEXUS_REPO = 'http://your-nexus-repo/repository/maven-releases/' // Nexus Repository URL
        DOCKER_IMAGE_NAME = 'your-image-name'
        DOCKER_REGISTRY = 'your-docker-registry-url'
        EKS_CLUSTER_NAME = 'your-eks-cluster'
        ARGOCD_APP_NAME = 'your-argocd-app'
        EMAIL_RECIPIENT = 'vasanthapuvvada2000@gmail.com' // Change to the desired recipient
    }

    stages {
        stage('Clone Repository') {
            steps {
                // Clone the Git repository
                git url: "${GIT_REPO_URL}", branch: 'main'
            }
        }

        stage('Build with Maven') {
            steps {
                // Use Maven to build the project and create the artifact
                script {
                    sh "'${MAVEN_HOME}/bin/mvn' clean install -DskipTests"
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Perform SonarQube analysis
                script {
                    withSonarQubeEnv(SONARQUBE) {
                        sh "'${MAVEN_HOME}/bin/mvn' sonar:sonar"
                    }
                }
            }
        }

        stage('Upload Artifact to Nexus') {
            steps {
                // Upload the artifact to Nexus repository
                script {
                    sh "'${MAVEN_HOME}/bin/mvn' deploy -DaltDeploymentRepository=nexus::default::${NEXUS_REPO}"
                }
            }
        }

        stage('Containerize the Artifact') {
            steps {
                // Build Docker image for the artifact
                script {
                    sh 'docker build -t ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} .'
                    sh 'docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}'
                }
            }
        }

        stage('Deploy to EKS using ArgoCD') {
            steps {
                // Deploy the artifact to EKS via ArgoCD
                script {
                    // Assuming ArgoCD CLI is set up and the Kubernetes context is configured
                    sh 'kubectl config use-context ${EKS_CLUSTER_NAME}'
                    sh 'argocd app sync ${ARGOCD_APP_NAME}'
                    sh 'argocd app wait ${ARGOCD_APP_NAME} --sync'
                }
            }
        }

    }

    post {
        success {
            // Send success email notification
            mail to: "${EMAIL_RECIPIENT}",
                 subject: "Jenkins Pipeline Successful - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "The pipeline has successfully completed the process."
        }

        failure {
            // Send failure email notification
            mail to: "${EMAIL_RECIPIENT}",
                 subject: "Jenkins Pipeline Failed - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "The pipeline has failed. Please check the logs for details."
        }

        always {
            // Archive the build artifacts
            archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
        }
    }
}
