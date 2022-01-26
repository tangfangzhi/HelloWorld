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
        echo "// git clean -dfx"
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
        echo "// git clean -dfx"
        mkdir -p debug
        cd debug
        cmake .. -DBUILD_HTTP=false -DBUILD_TOOLS=true > /dev/null
        make -j8> /dev/null
        echo "make install > /dev/null"
        cd ${WKC}/tests
        echo "pip3 install ${WKC}/src/connector/python/"
    '''
    return 1
}
pipeline {
    agent none
    options { skipDefaultCheckout() }
    environment{
        WK = '/root/jenkins/workspace/TDinternal'
        WKC = '/root/jenkins/workspace/TDinternal/community'
        LOGDIR = '/root/jenkins/workspace/log'
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
                stage ('build worker02') {
                    agent {label " worker02 "}
                    steps {
                        pre_test()
                        timeout(time: 100, unit: 'MINUTES') {
                            script {
                                sh '''
                                    echo "worker02 build done"
                                    date
                                '''
                            }
                        }
                    }
                }
                stage ('build worker03') {
                    agent {label " worker03 "}
                    steps {
                        pre_test()
                        timeout(time: 100, unit: 'MINUTES') {
                            script {
                                sh '''
                                    echo "worker03 build done"
                                    date
                                '''
                            }
                        }
                    }
                }
                stage ('build worker04') {
                    agent {label " worker04 "}
                    steps {
                        pre_test()
                        timeout(time: 100, unit: 'MINUTES') {
                            script {
                                sh '''
                                    echo "worker04 build done"
                                    date
                                '''
                            }
                        }
                    }
                }
                stage ('build worker05') {
                    agent {label " worker05 "}
                    steps {
                        pre_test()
                        timeout(time: 100, unit: 'MINUTES') {
                            script {
                                sh '''
                                    echo "worker05 build done"
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
                        time ./run.sh -m m.json -t tmp.task -l ${LOGDIR}
                        date
                    '''
                }
            }    
        }
    }
    post {
        success {
            emailext (
                subject: "PR-result: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' SUCCESS",
                body: """<!DOCTYPE html>
                    <html>
                        <head>
                            <meta charset="UTF-8">
                        </head>
                        <body leftmargin="8" marginwidth="0" topmargin="8" marginheight="4" offset="0">
                            <table width="95%" cellpadding="0" cellspacing="0" style="font-size: 16pt; font-family: Tahoma, Arial, Helvetica, sans-serif">
                                <tr>
                                    <td>
                                        <br/>
                                        <b><font color="#0B610B"><font size="6">构建信息</font></font></b>
                                        <hr size="2" width="100%" align="center" />
                                     </td>
                                </tr>
                                <tr>
                                    <td>
                                        <ul>
                                            <div style="font-size:18px">
                                                <li>构建名称>>分支：${env.BRANCH_NAME}</li>
                                                <li>构建结果：<span style="color:green"> Successful </span></li>
                                                <li>构建编号：${BUILD_NUMBER}</li>
                                                <li>触发用户：${env.CHANGE_AUTHOR}</li>
                                                <li>提交信息：${env.CHANGE_TITLE}</li>
                                                <li>构建地址：<a href=${BUILD_URL}>${BUILD_URL}</a></li>
                                                <li>构建日志：<a href=${BUILD_URL}console>${BUILD_URL}console</a></li>
                                            </div>
                                        </ul>
                                    </td>
                                </tr>
                            </table>
                        </body>
                    </html>""",
                to: "fztang@taosdata.com",
                from: "support@taosdata.com"
            )
        }
        failure {
            emailext (
                subject: "PR-result: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' FAIL",
                body: """<!DOCTYPE html>
                    <html>
                        <head>
                            <meta charset="UTF-8">
                        </head>
                        <body leftmargin="8" marginwidth="0" topmargin="8" marginheight="4" offset="0">
                            <table width="95%" cellpadding="0" cellspacing="0" style="font-size: 16pt; font-family: Tahoma, Arial, Helvetica, sans-serif">
                                <tr>
                                    <td>
                                        <br/>
                                        <b><font color="#0B610B"><font size="6">构建信息</font></font></b>
                                        <hr size="2" width="100%" align="center" />
                                    </td>
                                </tr>
                                <tr>
                                    <td>
                                        <ul>
                                            <div style="font-size:18px">
                                                <li>构建名称>>分支：${env.BRANCH_NAME}</li>
                                                <li>构建结果：<span style="color:red"> Failure </span></li>
                                                <li>构建编号：${BUILD_NUMBER}</li>
                                                <li>触发用户：${env.CHANGE_AUTHOR}</li>
                                                <li>提交信息：${env.CHANGE_TITLE}</li>
                                                <li>构建地址：<a href=${BUILD_URL}>${BUILD_URL}</a></li>
                                                <li>构建日志：<a href=${BUILD_URL}console>${BUILD_URL}console</a></li>
                                            </div>
                                        </ul>
                                    </td>
                                </tr>
                            </table>
                        </body>
                    </html>""",
                to: "fztang@taosdata.com",
                from: "support@taosdata.com"
            )
        }
    }
}
