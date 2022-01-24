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
      steps {
        echo "HelloWorld"
        sh '''
            date
            pwd
        '''
      }
    }
  }
}
