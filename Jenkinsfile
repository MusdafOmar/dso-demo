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
                    argocd app sync devsecops-demo \
                      --server argocd-server.argocd.svc.cluster.local:443 \
                      --auth-token "$ARGOCD_TOKEN" \
                      --insecure
                '''
            }
        }
    }
}
