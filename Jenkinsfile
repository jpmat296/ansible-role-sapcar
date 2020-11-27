import groovy.transform.Field
import org.jenkinsci.plugins.workflow.steps.FlowInterruptedException

@Field def ROLE_NAME = "ansible-role-sapcar"
@Field def GIT_URL = "git@github.com:jpmat296/${ROLE_NAME}.git"

stage('root') {
  try {
    parallel win2012r2: {
      stage('win2012r2') {
        node('vsphere') {
          sleep 1
          executeToxForPlatform('win2012r2')
        }
      }
    },
    win2016: {
      stage('win2016') {
        node('vsphere') {
          sleep 30
          executeToxForPlatform('win2016')
        }
      }
    },
    win2019: {
      stage('win2019') {
        node('vsphere') {
          sleep 60
          executeToxForPlatform('win2019')
        }
      }
    }
  } catch (e) {
    currentBuild.result = 'FAILURE'
    notifyFailed()
    throw e
  } finally {
    println 'currentBuild.result: ' + currentBuild.result
    if (currentBuild.result != 'FAILURE') {
      notifySuccessful()
    }
  }
}

def executeToxForPlatform(String platform) {
  checkout([$class: 'GitSCM', branches: [[name: env.BRANCH_NAME]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: ROLE_NAME]],                submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'f37e5623-35de-4909-ba30-72a3dad7a582', url: GIT_URL]]])
  stage('tox') {
    timestamps {
      try {
        stage('create') {
          sh """#!/bin/bash
            set -xe
            bash ~/bin/vms_destroy.sh || true
            source /usr/local/pyenv/.pyenvrc
            cd $ROLE_NAME
            tox -c tox-vsphere.ini --workdir .tox -e py38-$platform-sapcar-vsphere-createonly
          """
        }
        stage('test') {
          sh """#!/bin/bash
            set -xe
            source /usr/local/pyenv/.pyenvrc
            cd $ROLE_NAME
            tox -c tox-vsphere.ini --workdir .tox -e py38-$platform-sapcar-vsphere-testonly
          """
        }
      } finally {
        stage('destroy') {
          sh """#!/bin/bash
            set -xe
            source /usr/local/pyenv/.pyenvrc
            cd $ROLE_NAME
            tox -c tox-vsphere.ini --workdir .tox -e py38-$platform-sapcar-vsphere-destroyonly
            bash ~/bin/vms_destroy.sh || true
          """
        }
      }
    }
  }
}

def notifySuccessful() {
  emailext (
      subject: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",

body: """<p>SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>""",
      to: 'jp@matsusoft.com'
  )
}

def notifyFailed() {
  emailext (
      subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>""",
      to: 'jp@matsusoft.com'
  )
}
