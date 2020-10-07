import org.jenkinsci.plugins.workflow.steps.FlowInterruptedException

stage('root') {
  try {
    parallel win2012r2: {
      stage('win2012r2') {
        node('nomaster') {
           sleep 1
           executeMoleculeScenario('default')
        }
      }
    },
    win2016: {
      stage('win2016') {
        node('nomaster') {
          sleep 30
          executeMoleculeScenario('win2016')
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

def executeMoleculeScenario(String scenario) {
  checkout([$class: 'GitSCM', branches: [[name: env.BRANCH_NAME]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'sapcar']],                submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'f37e5623-35de-4909-ba30-72a3dad7a582', url: 'git@github.com:jpmat296/ansible-role-sapcar.git']]])
  stage('molecule') {
    timestamps {
      try {
        stage('create') {
          sh """#!/bin/bash
            set -xe
            bash ~/bin/vms_destroy.sh || true
            source /usr/local/pyenv/.pyenvrc
            cd sapcar
            pipenv sync
            export PATH=\$(pipenv --venv)/bin:\$PATH
            hash -r
            molecule create -s $scenario
          """
        }
        stage('test') {
          sh """#!/bin/bash
            set -xe
            source /usr/local/pyenv/.pyenvrc
            cd sapcar
            export PATH=\$(pipenv --venv)/bin:\$PATH
            hash -r
            molecule test --destroy never -s $scenario
          """
        }
      } finally {
        stage('destroy') {
          sh """#!/bin/bash
            set -xe
            source /usr/local/pyenv/.pyenvrc
            cd sapcar
            export PATH=\$(pipenv --venv)/bin:\$PATH
            molecule destroy -s $scenario
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
