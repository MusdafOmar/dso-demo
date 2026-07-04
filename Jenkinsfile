pipeline {
    agent {
        kubernetes {
            yamlFile 'build-agent.yaml'
        }
    }

    stages {
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Docker BnP') {
            steps {
                container('kaniko') {
                    sh '''
                    /kaniko/executor \
                      --context `pwd` \
                      --dockerfile Dockerfile \
                      --destination m2026/devsecops-demo:latest
                    '''
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                sh "echo 'done'"
            }
        }
    }
}
