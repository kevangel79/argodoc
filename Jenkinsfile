pipeline {
    agent { 
        docker { 
            image 'argo.registry:5000/debian-jessie-argodoc' 
        }
    }
    options {
        checkoutToSubdirectory('argodoc')
        newContainerPerStage()
    }
    environment {
        RPOJECT_NAME="argodoc"
        GIT_COMMITTER_NAME="newgrnetci"
        GIT_COMMITTER_EMAIL="<>"

    }
    stages {
        stage('Build') {
            when { branch pattern: "master|devel|PR-.*", comparator: "REGEXP" }
            parallel {
                stage ('Build web-api docs'){
                    steps {
                        dir ("${WORKSPACE}/argo-web-api") {
                            git branch: "${env.BRANCH_NAME}",
                                credentialsId: 'jenkins-rpm-repo',
                                url: 'git@github.com:ARGOeu/argo-web-api.git'
                            sh """
                                cd ${WORKSPACE}/argo-web-api/doc/v2
                                mkdocs build --clean
                                cp -R ${WORKSPACE}/argo-web-api/doc/doc/ ${WORKSPACE}/argodoc/content/guides
                                cp -R ${WORKSPACE}/argo-web-api/doc/v2/site/ ${WORKSPACE}/argodoc/api/v2
                                cd ${WORKSPACE}/argodoc
                                if [ -n "\$(git status --porcelain)" ]; then
                                    git add -A
                                    git commit -a --author="newgrnetci <>" -m "Update argo-web-api docs"
                                    #git push origin devel
                                fi
                            """
                            //deleteDir()
                        }
                    }
                }
                stage ('Build messaging docs'){
                    steps {
                        dir ("${WORKSPACE}/argo-messaging") {
                            git branch: "${env.BRANCH_NAME}",
                                credentialsId: 'jenkins-rpm-repo',
                                url: 'git@github.com:ARGOeu/argo-messaging.git'
                            sh """
                                cd ${WORKSPACE}/argo-messaging/doc/v1
                                mkdocs build --clean
                                cp -R ${WORKSPACE}/argo-messaging/doc/doc/ ${WORKSPACE}/argodoc/content/guides
                                cp -R ${WORKSPACE}/argo-messaging/doc/v1/site/* ${WORKSPACE}/argodoc/messaging/v1
                                cd ${WORKSPACE}/argodoc
                                if [ -n "\$(git status --porcelain)" ]; then
                                    git add -A
                                    git commit -a --author="newgrnetci <>" -m "Update argo-messaging docs"
                                    #git push origin devel
                                fi
                            """
                            //deleteDir()
                        }
                    }
                }
                stage ('Build authn docs'){
                    steps {
                        dir ("${WORKSPACE}/argo-api-authn") {
                            git branch: "${env.BRANCH_NAME}",
                                credentialsId: 'jenkins-rpm-repo',
                                url: 'git@github.com:ARGOeu/argo-api-authn.git'
                            sh """
                                cd ${WORKSPACE}/argo-api-authn/docs/v1
                                mkdocs build --clean
                                cp -R ${WORKSPACE}/argo-api-authn/docs/v1/docs/ ${WORKSPACE}/argodoc/content/guides
                                cp -R ${WORKSPACE}/argo-api-authn/docs/v1/site/* ${WORKSPACE}/argodoc/authn/v1
                                cd ${WORKSPACE}/argodoc
                                if [ -n "\$(git status --porcelain)" ]; then
                                    git add -A
                                    git commit -a --author="newgrnetci <>" -m "Update argo-authn docs"
                                    #git push origin devel
                                fi
                            """
                            //deleteDir()
                        }
                    }
                }
            }
        }
        stage('Deploy mkdocs') {
            when {
                branch "master|devel"
            }
            steps {
                echo 'Deploying mkdocs...'
                dir ("${WORKSPACE}/argoeu") {
                    git branch: "${env.BRANCH_NAME}",
                        credentialsId: 'jenkins-rpm-repo',
                        url: 'git@github.com:ARGOeu/argoeu.github.io.git'
                    sh """
                        rm -rf ${WORKSPACE}/argoeu/api
                        rm -rf ${WORKSPACE}/argoeu/messaging
                        rm -rf ${WORKSPACE}/argoeu/authn
                        cp -R ${WORKSPACE}/argodoc/api ${WORKSPACE}/argoeu
                        cp -R ${WORKSPACE}/argodoc/messaging ${WORKSPACE}/argoeu
                        cp -R ${WORKSPACE}/argodoc/authn ${WORKSPACE}/argoeu
                        cd ${WORKSPACE}/argoeu
                        if [ -n "\$(git status --porcelain)" ]; then
                            git add -A
                            git commit -a --author="newgrnetci <>" -m "Update docs"
                            #git push origin master
                        fi
                    """
                    //deleteDir()
                }
            }
        } 
    }
}