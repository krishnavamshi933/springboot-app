pipeline {
    agent any

    parameters {
        string(name: 'gitCredentialsId', description: 'Git Credentials ID', defaultValue: 'git')
        string(name: 'sonarProjectKey', description: 'SonarQube Project Key', defaultValue: 'sample-ecr')
        string(name: 'sonarProjectName', description: 'SonarQube Project Name', defaultValue: 'sample-ecr')
        string(name: 'sonarHostUrl', description: 'SonarQube Host URL', defaultValue: 'http://3.0.103.126:9000')
        string(name: 'sonarLogin', description: 'SonarQube Login Token', defaultValue: 'sqp_2cfedf4569c1554828f8c3bfec5ea4e1e0a076a8')
        string(name: 'awsCredentialsId', description: 'AWS ECR Credentials ID', defaultValue: 'aws-ecr-credentials1')
        string(name: 'awsRegion', description: 'AWS Region', defaultValue: 'ap-southeast-1')
        string(name: 'ecrRepoName', description: 'ECR Repository Name', defaultValue: 'krishnavamshi')
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the source code from Git
                git branch: 'main', credentialsId: params.gitCredentialsId, url: 'git@github.com:krishnavamshi933/springboot-app.git'
            }
        }

        stage('Build') {
            steps {
                // Build the Maven project
                sh 'mvn clean install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Run SonarQube analysis
                sh "mvn sonar:sonar -Dsonar.projectKey=${params.sonarProjectKey} -Dsonar.projectName='${params.sonarProjectName}' -Dsonar.host.url=${params.sonarHostUrl} -Dsonar.login=${params.sonarLogin}"
            }
        }

        stage('Docker Build') {
            steps {
                // Build the Docker image
                sh 'docker build -t krishnavamshi .'
            }
        }

        stage('ECR Image Push') {
            steps {
                // Get the ECR login password using AWS CLI
                withCredentials([usernamePassword(credentialsId: params.awsCredentialsId, usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh "aws ecr get-login-password --region ${params.awsRegion} | docker login --username AWS --password-stdin .amazonaws.com"
                }

                // Tag the Docker image
                sh "docker tag krishnavamshi:latest 074469624262.dkr.ecr.ap-southeast-1.amazonaws.com/${params.ecrRepoName}:latest"

                // Push the Docker image to AWS ECR
                sh "docker push 074469624262.dkr.ecr.ap-southeast-1.amazonaws.com/${params.ecrRepoName}:latest"
            }
        }
    }
}
