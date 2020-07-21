import org.jenkinsci.plugins.workflow.steps.FlowInterruptedException

stage('root') {
  try {
    parallel win2012r2: {
      stage('win2012r2') {
        node('lp009') {
           sleep 1
           executeMoleculeScenario('default')
        }
      }
    },
    win2016: {
      stage('win2016') {
        node('lp009') {
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
  checkout([$class: 'GitSCM', branches: [[name: '*/master']],      doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'jpmat296.localization']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'f37e5623-35de-4909-ba30-72a3dad7a582', url: 'git@github.com:jpmat296/ansible.localization.git']]])
  stage('molecule') {
    timestamps {
      try {
        sh """#!/bin/bash
          set -x
          bash ~/bin/vms_destroy.sh
          cd sapcar
          molecule test -s $scenario
        """
      } catch (FlowInterruptedException ie) {
        sh """#!/bin/bash
          set -x
          cd sapcar
          molecule destroy -s $scenario
        """
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
