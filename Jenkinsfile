// import shared library 
@Library('jenkins-shared-library') _


pipeline {
    agent none
    stages {
        stage('Check bash syntax') {
            agent { docker { image 'koalaman/shellcheck-alpine:latest' } }
            steps {

              script { syntaxChecking.bashCheck }

            }
        }
        stage('Check yaml syntax') {
            agent { docker { image 'sdesbure/yamllint' } }
            steps {
              script { syntaxChecking.yamlCheck }
            }
        }
         stage('Check markdown syntax') {
            agent { docker { image 'ruby:alpine' } }
            steps {
              script { syntaxChecking.markdownCheck }
            }
         }
         stage('Prepare ansible environment') {
            agent any
            environment {
                VAULTKEY = credentials('vaultkey')
                DEVOPSKEY = credentials('devopskey')
            }
            steps {
                sh 'echo \$VAULTKEY > vault.key'
                sh 'cp \$DEVOPSKEY id_rsa'
                sh 'chmod 600 id_rsa'
            }
         }
         stage('Test and deploy the application') {
            agent { docker { image 'registry.gitlab.com/robconnolly/docker-ansible:latest' } }
            stages {
               stage("Install ansible role dependencies") {
                   steps {
                       sh 'ansible-galaxy install -r roles/requirements.yml'
                   }
               }
                stage("Ping targeted hosts") {
                   steps {
                       sh 'ansible all -m ping -i hosts --private-key id_rsa'
                   }
                }
                 stage("Vérify ansible playbook syntax") {
                   steps {
                       sh 'ansible-lint -x 306 install_student_list.yml'
                       sh 'echo "${GIT_BRANCH}"'
                   }
                 } 
               stage("Build docker images on build host") {
                   when {
                      expression { GIT_BRANCH == 'origin/master' }
                   }
                   steps {
                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --tags "build" --limit build install_student_list.yml'
                   }
               }
               stage("Deploy app in production") {
                    when {
                       expression { GIT_BRANCH == 'origin/master' }
                    }
                   steps {
                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --tags "deploy" --limit prod install_student_list.yml'
                   }
               }
            }
         }
    }
    post {
     always {
       script {
         // Use slackNotifier.groovy from shared library and provide current build result as parameter 

         clean
         slackNotifier currentBuild.result
     }
    }

}


