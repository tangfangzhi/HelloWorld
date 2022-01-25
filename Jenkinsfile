import hudson.model.Result
import hudson.model.*;
import jenkins.model.CauseOfInterruption
node {
}

pipeline {
  agent none
  options { skipDefaultCheckout() }
  environment{
    WK = '/var/lib/jenkins/workspace/TDinternal'
  }
  stages {
    stage('pre_build'){
      agent{label " slave1 || slave11 "}
      steps {
        echo "HelloWorld"
        echo "BRANCH: ${env.BRANCH_NAME}"
        sh '''
            date
            pwd
            env
        '''
      }
    }
  }
}
