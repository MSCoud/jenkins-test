#!/usr/bin/env groovy


def docker_image_name = ''
def version = ''

node {
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

//
//
//    docker.image('trion/ng-cli:12.2.12').inside {
//
//
//    }
//
//    def dockerImage
//    stage('build docker') {
//    }
//
//    stage('publish and Archiving') {
//     }

}
