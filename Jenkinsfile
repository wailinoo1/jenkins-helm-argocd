pipeline {
    agent none

    stages {
        stage('Git Clone') {
            agent any
            steps {
                git branch: 'main', credentialsId: '041d1fea-8d62-49e3-84a2-d582ec3c08e4', url: 'https://gitlab.com/Wai_Lin_Oo/ci-code-repo.git'
                echo "Hello"
            }
        }
        stage("Build & Push"){
            agent any
            steps {
                withCredentials([usernamePassword(credentialsId: '932ba7c2-2cb1-46a7-b10c-ffb9628fe3ae', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                sh 'docker login -u $USERNAME -p $PASSWORD'
                sh 'docker build -t wailinoo/cirepo:${BUILD_ID} .'
                sh 'docker push wailinoo/cirepo:${BUILD_ID}'      
                sh 'docker rmi -f $(docker images -aq)'
                }
            }
        }
        stage("Update Manifest"){
            agent{
                docker { image 'ubuntu'
                         args '-u root -v /var/run/docker.sock:/var/run/docker.sock'
                         reuseNode true
                }
            }
            environment{
                GITLAB_CREDENTIAL_CD = credentials('GITLAB_CREDENTIAL_CD')
            }
                steps{
                    withCredentials([usernamePassword(credentialsId: 'GITLAB_CREDENTIAL_CD', usernameVariable: 'USERNAME', passwordVariable: 'TOKEN')]){
                    script{
                        sh 'apt-get update && apt-get install git -y'
                        sh 'git config --global user.email "wailinoo2012@gmail.com"'
                        sh 'git config --global user.name wailinoo'
                        sh '[ -d cd-manifest-repo ] && rm -rf cd-manifest-repo || echo "Directory does not exist , skipping removal"'

                        sh 'git clone https://${USERNAME}:${TOKEN}@gitlab.com/${USERNAME}/cd-manifest-repo.git'
                        dir('cd-manifest-repo') {
                        sh 'ls -lrt'
                        sh 'git config --global --add safe.directory /var/lib/jenkins/workspace/gitlab-jenkins-ci'
                        sh 'sed -i "s|wailinoo/cirepo:[^[:space:]]*|wailinoo/cirepo:${BUILD_ID}|g" values.yaml'
                        sh 'git add values.yaml'
                        sh 'git commit -am "update values.yaml file"'
                        sh 'git push -u origin main'
                        }
                    }
                }
                }
        }
    }
}
