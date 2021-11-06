#!/usr/bin/env groovy


def docker_image_name = ''
def version = ''

node ("APPLI-ETIXO-04") {
    stage('Checkout') {
        checkout scm
    }

    stage('Display aop infos') {
      props = readJSON file: 'package.json'
      echo "from gitlab gitlabBranch ${env.gitlabBranch}"
      echo "from gitlab gitlabSourceBranch ${env.gitlabSourceBranch}"
      echo "from gitlab gitlabActionType ${env.gitlabActionType}"
      echo "app name ${props.name}"
      echo "app version ${props.version}"
      docker_image_name = props.name
      version = props.version
      updateGitlabCommitStatus name: "${docker_image_name}:v${version}", state: 'pending'

    }



    docker.image('trion/ng-cli:12.2.12').inside {
      stage('Check ng-cli and node version ') {
        echo "afficher les version angular-cli"
        sh "ng version"
        sh "node --version"
      }

      stage('Install Dependencies ') {
        sh "npm install"
      }

//
//      stage('Test unitaire ') {
//        sh "ng test"
//      }

      stage('Packaging ') {
        sh "rm -rf build/"
        sh "ng build"
      }

    }

    def dockerImage
    stage('build docker') {
      sh "mkdir build/"
      sh "cp -R docker build/"
      sh "cp -R dist build/docker/"
      dockerImage = docker.build("sandbox/${docker_image_name}:v${version}", 'build/docker')
    }


    stage('Archiving') {
      //      to be archived
      sh "docker save -o ${docker_image_name}:v${version}.tar  sandbox/${docker_image_name}:v${version}"
      archiveArtifacts artifacts: "${docker_image_name}:v${version}.tar", fingerprint: true

      //      to be used for deployement en dev env
      sh "docker tag sandbox/${docker_image_name}:v${version}  sandbox/${docker_image_name}:dev"
      sh "docker save -o ${docker_image_name}-dev.tar  sandbox/${docker_image_name}:dev"


      updateGitlabCommitStatus name: "${docker_image_name}:v${version}", state: 'success'
    }


  stage('Deployment') {

    sshagent(['ci.etixway']) {
      sh "scp -o StrictHostKeyChecking=no ${docker_image_name}-dev.tar  root@10.10.40.98:${docker_image_name}-dev.tar "
      sh 'ssh -o StrictHostKeyChecking=no root@10.10.40.98 docker-compose -f /docker-workspace/sandbox/jenkins-project/docker-compose.yml down'
//      sh "ssh -o StrictHostKeyChecking=no root@10.10.40.98 docker image rm ${docker_image_name}:dev "
      sh "ssh -o StrictHostKeyChecking=no root@10.10.40.98 docker load -i ${docker_image_name}-dev.tar"
      sh 'ssh -o StrictHostKeyChecking=no root@10.10.40.98 docker-compose -f /docker-workspace/sandbox/jenkins-project/docker-compose.yml up -d'
    }
  }

//    def dockerImage
//    stage('build docker') {
//    }
//
//    stage('publish and Archiving') {
//     }

}
