#!/usr/bin/env groovy

def imageName = 'jenkinsciinfra/404'

properties([
    buildDiscarder(logRotator(numToKeepStr: '5', artifactNumToKeepStr: '5')),
    pipelineTriggers([[$class:"SCMTrigger", scmpoll_spec:"H/15 * * * *"]]),
])

node('docker&&linux') {
    def container
    stage('Build') {
        timestamps {
            checkout scm
            sh 'git rev-parse HEAD > GIT_COMMIT'
            shortCommit = readFile('GIT_COMMIT').take(6)
            def imageTag = "${env.BUILD_ID}-build${shortCommit}"
            echo "Creating the container ${imageName}:${imageTag}"
            container = docker.build("${imageName}:${imageTag}")
        }
    }

    /* Assuming we're not inside of a pull request or multibranch pipeline */
    if (infra.isTrusted()) {
        stage('Publish') {
            infra.withDockerCredentials {
                timestamps { container.push() }
            }
        }
    }
}
