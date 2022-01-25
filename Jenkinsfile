import hudson.model.Result
import hudson.model.*;
import jenkins.model.CauseOfInterruption
node {
}
def sync_source() {
    sh'hostname'
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
        } else if (env.CHANGE_TARGET == '2.0') {
            sh '''
                cd ${WKC}
                git checkout 2.0
            '''
        } else {
            sh '''
                cd ${WKC}
                git checkout develop
            '''
        }
    }
    sh'''
        cd ${WKC}
        git remote prune origin
        git pull >/dev/null
        git fetch origin +refs/pull/${CHANGE_ID_TMP}/merge
        git checkout -qf FETCH_HEAD
        // git clean -dfx
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
        } else if (env.CHANGE_TARGET == '2.0') {
            sh '''
                cd ${WK}
                git checkout 2.0
            '''
        } else {
            sh '''
                cd ${WK}
                git checkout develop
            '''
        }
    }
}
def pre_test() {
    sync_source()
    sh '''
        cd ${WK}
        git pull >/dev/null
        export TZ=Asia/Harbin
        date
        // git clean -dfx
        mkdir debug
        cd debug
        cmake .. -DBUILD_HTTP=false -DBUILD_TOOLS=true > /dev/null
        make -j8> /dev/null
        make install > /dev/null
        cd ${WKC}/tests
        pip3 install ${WKC}/src/connector/python/
    '''
    return 1
}
pipeline {
    agent none
    options { skipDefaultCheckout() }
    environment{
        WK = '/root/jenkins/workspace/TDinternal'
        WKC = '/root/jenkins/workspace/TDinternal/community'
        CHANGE_ID_TMP = '9982'
    }
    stages {
        stage ('pre_build') {
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
        stage ('Parallel build stage') {
            //only build pr
            options { skipDefaultCheckout() }
            when {
                allOf {
                    changeRequest()
                    not { expression { env.CHANGE_BRANCH =~ /docs\// }}
                }
            }
            parallel {
                stage ('dispatcher sync source') {
                    agent {label " dispatcher "}
                    steps {
                        sync_source()
                        timeout(time: 100, unit: 'MINUTES') {
                            script {
                                sh '''
                                    echo "dispatcher ready"
                                    date
                                '''
                            }
                        }
                    }
                }
                stage ('build worker01') {
                    agent {label " worker01 "}
                    steps {
                        pre_test()
                        timeout(time: 100, unit: 'MINUTES') {
                            script {
                                sh '''
                                    echo "worker01 build done"
                                    date
                                '''
                            }
                        }
                    }
                }
            }
        }
        stage('run test') {
            agent {label " dispatcher "}
            steps {
                timeout(time: 100, unit: 'MINUTES'){
                    sh '''
                        date
                        cd ${WKC}/tests/parallel_test
                        time ./run.sh -m m.json -t tmp.task
                        date
                    '''
                }
            }    
        }
    }
}
