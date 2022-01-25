import hudson.model.Result
import hudson.model.*;
import jenkins.model.CauseOfInterruption
node {
}
def pre_test(){
    sh 'hostname'
    sh ''' 
        cd ${WKC}
        git reset --hard HEAD~10 >/dev/null
    ''' 
    script {
        if (env.CHANGE_TARGET == 'master') {
            sh ''' 
                cd ${WKC}
                git checkout master
            '''
        } else if (env.CHANGE_TARGET == '2.4') {
            sh ''' 
                cd ${WKC}
                git checkout 2.4 
            '''
        } else {
            sh ''' 
                cd ${WKC}
                git checkout develop
            '''
        }   
    }   
    sh '''
        cd ${WKC}
        git remote prune origin
        [ -f src/connector/grafanaplugin/README.md ] && rm -f src/connector/grafanaplugin/README.md > /dev/null || echo "failed to remove grafanaplugin README.md"
        git pull >/dev/null
        git clean -dfx
        git submodule update --init --recursive
        cd ${WK}
        git reset --hard HEAD~10
    ''' 
    script {
        if (env.CHANGE_TARGET == 'master') {
            sh ''' 
                cd ${WK}
                git checkout master
            '''
        } else if(env.CHANGE_TARGET == '2.4') {
            sh '''
                cd ${WK}
                git checkout 2.4
            '''
        } else {
            sh '''
                cd ${WK}
                git checkout develop
            '''
        }
    }
    sh '''
        cd ${WK}
        git pull >/dev/null
        git fetch origin +refs/pull/${CHANGE_ID_TMP}/merge
        git checkout -qf FETCH_HEAD
        export TZ=Asia/Harbin
        date
        git clean -dfx
        mkdir debug
        cd debug
        cmake .. -DBUILD_HTTP=false -DBUILD_TOOLS=true > /dev/null
        make > /dev/null
        make install > /dev/null
    '''
    return 1
}

pipeline {
    agent none
    options { skipDefaultCheckout() }
    environment{
        WK = '/root/jenkins/workspace/TDinternal'
        WKC = '/root/jenkins/workspace/TDinternal/community'
        CHANGE_ID_TMP = '623'
    }
    stages {
        stage('pre_build'){
            agent{label " dispatcher "}
            steps {
                sh '''
                    date
                    pwd
                    env
                    hostname
                '''
            }
        }
        stage('Parallel build stage') {
            //only build pr
            options { skipDefaultCheckout() }
            when {
                allOf {
                    changeRequest()
                    not { expression { env.CHANGE_BRANCH =~ /docs\// }}
                }
            }
            parallel {
                stage('build worker01') {
                    agent{label " worker01 "}
                    steps {
                        pre_test()
                        timeout(time: 100, unit: 'MINUTES') {
                            script {
                                scope.each {
                                    sh """
                                        date
                                    """
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}
