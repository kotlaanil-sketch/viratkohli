pipeline {
    agent any

    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    environment {
        SONAR_SERVER = 'SonarServer'
        SONAR_PROJECT_KEY = 'EcommerceApp'
        GIT_REPO = 'https://github.com/kotlaanil-sketch/viratkohli.git'
        EMAIL_TO = 'yourmail@gmail.com'
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Checkout Code') {
            steps {
                git url: "${GIT_REPO}", branch: 'main'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONAR_SERVER}") {
                    sh """
                    mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=${SONAR_PROJECT_KEY}
                    """
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh '''
                cp target/*.war /var/lib/tomcat9/webapps/
                '''
            }
        }

        stage('Upload Sonar Report to GitHub') {
            steps {
                sh '''
                curl -u admin:<SONAR_TOKEN> \
                "http://<SONAR_EC2_IP>:9000/api/issues/search?componentKeys=${SONAR_PROJECT_KEY}" \
                -o sonar-report.json

                git config user.email "ci@jenkins.com"
                git config user.name "Jenkins CI"
                git add sonar-report.json
                git commit -m "Added SonarQube analysis report"
                git push origin main
                '''
            }
        }
    }

    post {
        success {
            emailext(
                subject: "SUCCESS: ${JOB_NAME} #${BUILD_NUMBER}",
                body: """
                Job: ${JOB_NAME}
                Build: ${BUILD_NUMBER}
                Status: SUCCESS
                Time: ${BUILD_TIMESTAMP}
                """,
                to: "${EMAIL_TO}"
            )
        }

        failure {
            emailext(
                subject: "FAILED: ${JOB_NAME} #${BUILD_NUMBER}",
                body: """
                Job: ${JOB_NAME}
                Build: ${BUILD_NUMBER}
                Status: FAILED
                Time: ${BUILD_TIMESTAMP}
                """,
                to: "${EMAIL_TO}"
            )
        }
    }
}
