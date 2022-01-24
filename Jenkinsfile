import hudson.model.Result
import hudson.model.*;
import jenkins.model.CauseOfInterruption
node {
}

pipeline {
  agent none
  options { skipDefaultCheckout() }
  environment{
    WKDIR="/data/tmp"
  }
  stages {
    stage('pre_build'){
      agent{label 'prebuild'}
      options { skipDefaultCheckout() }
      when {
        changeRequest()
      }
      steps {
        pwd
        echo "HelloWorld"
      }
    }
  }
}
