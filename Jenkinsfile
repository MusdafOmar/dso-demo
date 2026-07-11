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
        stage('SCA') {
            steps {
                container('maven') {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        sh 'mvn org.owasp:dependency-check-maven:check'
                    }
                }
            }
            post {
                always {
                    archiveArtifacts allowEmptyArchive: true, artifacts: 'target/dependency-check-report.html', fingerprint: true
                }
            }
        }
        stage('Generate SBOM') {
            steps {
                container('maven') {
                    sh 'mvn org.cyclonedx:cyclonedx-maven-plugin:makeAggregateBom'
                }
            }
            post {
                success {
                    dependencyTrackPublisher projectName: 'demo', projectVersion: '0.0.1-SNAPSHOT', artifact: 'target/bom.xml'
                    archiveArtifacts allowEmptyArchive: true, artifacts: 'target/bom.xml', fingerprint: true, onlyIfSuccessful: true
                }
            }
        }
        stage('OSS License Checker') {
            steps {
                container('licensefinder') {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        sh 'license_finder'
                    }
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
