pipeline {
    agent { node { label "slave1" }}
    options {
           buildDiscarder(logRotator(numToKeepStr: '5'))
           disableConcurrentBuilds()
           skipDefaultCheckout false
           timeout(time: 30, unit: 'MINUTES')
           timestamps()
        }
    stages{
        stage('check out'){
            steps{
                git branch: 'main', url: 'https://github.com/govardhan992/dockerhub.git'
            }
        }
        stage('build the docker image'){
            steps {
                sh "docker build -t govardhanr992/nginx:${BUILD_NUMBER} ."
            }
        }
        stage("push docker image"){
            steps {
                withCredentials([string(credentialsId: 'dockerhub_pwd', variable: 'dockerhub_pwd')]) {
                
               sh "docker login -u govardhanr992 -p ${dockerhub_pwd}"
     }
                sh "docker push govardhanr992/nginx:${BUILD_NUMBER}"
            }
        }
        stage("deploy_app"){
            steps {
              script {

                 def USER_INPUT = input(
                    message: 'User input required - Do you want to proceed?',
                    parameters: [
                            [$class: 'ChoiceParameterDefinition',
                             choices: ['no','yes'].join('\n'),
                             name: 'input',
                             description: 'Menu - select box option']
                    ])
                    echo "The answer is: ${USER_INPUT}"
                    if( "${USER_INPUT}" == "yes"){
                sshagent(['deploy_container']) {
                 
                 sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.45.191 docker pull govardhanr992/nginx:${BUILD_NUMBER}'
                 sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.45.191 docker rm -f webserver || true'
                 sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.45.191 docker run -d -p 8090:8080 --name webserver govardhanr992/nginx:${BUILD_NUMBER}'
                }
                   
            }
                else {
                   echo "Skipping deployment"
                }
         }
            }
        }
    }
  post{

 success{
 emailext to: 'govardhanr992@gmail.com',
          subject: "Pipeline Build is over .. Build # is ..${env.BUILD_NUMBER} and Build status is.. ${currentBuild.result}.",
          body: "Pipeline Build is over .. Build # is ..${env.BUILD_NUMBER} and Build status is.. ${currentBuild.result}.",
          replyTo: 'govardhanr992@gmail.com'
 }
          failure{
 emailext to: 'govardhanr9922@gmail.com',
          subject: "Pipeline Build is over .. Build # is ..${env.BUILD_NUMBER} and Build status is.. ${currentBuild.result}.",
          body: "Pipeline Build is over .. Build # is ..${env.BUILD_NUMBER} and Build status is.. ${currentBuild.result}.",
          replyTo: 'govardhanr992@gmail.com'
 }
  always {
          cleanWs()
          }
 
}
        } // pipeline closing
