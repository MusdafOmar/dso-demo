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
                    dependencyTrackPublisher projectName: 'demo', projectVersion: '0.0.1-SNAPSHOT', artifact: 'target/bom.xml', synchronous: false
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
        stage('SAST') {
            steps {
                container('slscan') {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        sh 'scan --src . --type java'
                    }
                }
            }
            post {
                always {
                    archiveArtifacts allowEmptyArchive: true, artifacts: 'reports/*', fingerprint: true
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
                      --destination m2026/devsecops-demo:multistage
                    '''
                }
            }
        }
        stage('Image Analysis') {
            parallel {
                stage('Image Linting') {
                    steps {
                        container('dockle') {
                            sh 'dockle docker.io/m2026/devsecops-demo:multistage'
                        }
                    }
                }

                stage('Image Scan') {
                    steps {
                        container('trivy') {
                            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                                sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL m2026/devsecops-demo:multistage'
                            }
                        }
                    }
                }
            }
        }
                        stage('Deploy to Dev') {
            steps {
                sh "echo 'done'"
            }
        }

        stage('ArgoCD Sync') {
            steps {
                container('argocd') {
                    withCredentials([
                        string(
                            credentialsId: 'argocd-token',
                            variable: 'ARGOCD_TOKEN'
                        )
                    ]) {
                        sh '''
                            apk add --no-cache curl

                            curl -sSL -o /tmp/argocd \
                              https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64

                            chmod +x /tmp/argocd

                            /tmp/argocd app sync devsecops-demo \
                              --server argocd-server.argocd.svc.cluster.local:443 \
                              --auth-token "$ARGOCD_TOKEN" \
                              --insecure
                        '''
                    }
                }
            }
        }
    }
}          
