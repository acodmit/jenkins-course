// email function
def sendEmail(arg1, arg2, arg3, arg4) {
    echo "JOB_NAME = ${arg1}, BUILD_ID = ${arg2}, status = ${arg3}"
    echo "OUTPUT_STRING = ${arg4}"
}

def OUTPUT = ''
//def PASS = ''

// pipeline structure
pipeline {
    agent any
    parameters {
        string defaultValue: 'pipeline_default.zip', description: 'some description', name: 'test_artifact'
        booleanParam description: 'test boolean parameter', name: 'test_bool_param'
        booleanParam defaultValue: true, name: 'RUN_TEST'
          booleanParam defaultValue: true, name: 'SEND_EMAIL'
    }
    stages {
        stage('Download') {
            steps {
                cleanWs()
                echo (message:"Download1")
                dir ("pipeline") {
                    git branch: 'pipeline', url: 'https://github.com/KLevon/jenkins-course.git'
                }
                rtDownload (
                    serverId:'test_id',
                    spec: '''{
                        "files" : [
                            {
                            "pattern" : "generic-local/libraries/printer.zip",
                            "target" : "./",
                            "flat":"true"
                            }
                        ]
                        }'''
                )
                unzip (
                    zipFile: "printer.zip",
                    dir:"pipeline"
                )
            }
        }
        stage('Build') {
            steps {
                echo (message:"Build2")
                bat (
                    script:"""
                        cd pipeline
                        Makefile.bat
                    """
                )
            }
        }
        stage('Test') {
            when {
                equals expected: true,
                actual: params.RUN_TEST
            }
            steps {
                script {
                def array = ['printer', 'scanner', 'main']
                for (el in array) {
                    OUTPUT += bat (
                        script:"""
                            cd pipeline
                            tests.bat ${el}
                        """,
                        returnStdout: true
                    ).trim()
                }
            }
            }
        }
        stage('Dynamic') {
            when {
                branch "feature/test_branch2*"
            }
            echo "TEST BRANCH 2"
        }
        stage('Publish') {
            steps {
                echo (message:"Publish3")
            script {
                zip (zipFile:"${params.test_artifact}",
                    archive: true,
                    dir:"./pipeline",
                    glob:""
                )
            }
            echo ("RTUPLOAD")
            withCredentials (
                [usernamePassword(credentialsId:'user_acodmit', passwordVariable:'psw',usernameVariable: 'usr')])
                {
                    echo ("USERNAME: ${usr}, PASSWORD: ${psw}")
                    //PASS += "${psw}"
                    //echo PASS
                }
                rtUpload (
                    serverId:'test_id',
                    spec: """{
                        "files" : [
                            {
                            "pattern" : "${params.test_artifact}",
                            "target" : "generic-local/release/aleksandar/${env.BUILD_ID}/"
                            }
                        ]
                        }"""
                )
                //script {
                //    sendEmail(env.JOB_NAME, env.BUILD_ID, currentBuild.currentResult)
                //}
                script {
                    if (params.test_bool_param) {
                        error("PARAM FALSE -> EXIT")
                    }
                }
            }
        }
    }
    post {
        failure {
            sendEmail(env.JOB_NAME, env.BUILD_ID, currentBuild.currentResult, OUTPUT)
        }
    }
}
