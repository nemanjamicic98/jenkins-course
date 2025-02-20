def someFunction(arg1, arg2, arg3)
{
    echo "Build $arg1 for following $arg2 passed. Results $arg3."
}
def output = ""
pipeline {
    agent any
    parameters {
            string defaultValue: 'pipeline.zip', description: 'Output zip file name.', name: 'VAR_NAME'
            booleanParam defaultValue: false, name: 'FAIL_PIPELINE'
            string defaultValue: 'pipeline', name: 'GIT_BRANCH'
            booleanParam defaultValue: false, name: 'RUN_TEST'
            booleanParam defaultValue: false, name: 'SEND_EMAIL'
    }

    stages{
        stage('Download'){
            steps {
                cleanWs()
                echo (message: "Download step")
                dir('pypeline_folder'){
                  git(
                    branch: "${params.GIT_BRANCH}",
                    url: 'https://github.com/KLevon/jenkins-course'
                )
                }
                rtDownload(
                    serverId: 'Artifactory',
                    spec: '''{
                            "files": [
                            {
                                "pattern" : "generic-local/libraries/printer.zip",
                                "target": "nemanja/printer.zip"
                            }
                            ]
                        }''')
                unzip(
                    zipFile: "nemanja/libraries/printer.zip",
                    dir: "pypeline_folder/")
            }
        }
        stage('Build'){
            steps{
                echo (message: "Build step")
                withCredentials (
                    [usernamePassword(credentialsId: '011298', passwordVariable: 'psw', usernameVariable: 'usr')]
                    )
                    {
                        echo "$psw and $usr. "
                    }
                dir ("pypeline_folder"){
                    bat (
                        script: "./Makefile.bat")
                }
            }
        }
        stage('Tests'){
            when {
                equals expected: true,
                actual: params.RUN_TEST
            }
            steps{
                echo (message: "Tests step")
                dir ("pypeline_folder"){
                    script {
                        def array = ["printer", "scanner", "main"]
                        for (element in array)
                        {
                           output += bat (
                            script: "./Tests.bat ${element}", returnStdout: true).trim() 
                        }
                    }
                }
                someFunction("${env.BUILD_ID}", "${env.JOB_NAME}", output)
            }
        }
        stage('Publish'){
            steps{
                echo (message: "Publish step")
                script{
                    zip (
                        zipFile: "./pypeline_folder/nemanja.zip",
                        archieve: true,
                        dir: "./pypeline_folder/"
                        )
                }
                rtUpload(
                    serverId: 'Artifactory',
                    spec: """{
                            "files": [
                            {
                                "pattern" : "./pypeline_folder/nemanja.zip",
                                "target": "generic-local/nemanja/${env.BUILD_ID}/${VAR_NAME}"
                            }
                            ]
                        }""")
                script {

                    if (FAIL_PIPELINE == "true")
                    {
                        bat "exit 1"
                    }
                }
            }
        }
    }
    post {
        failure {
            someFunction("${env.BUILD_ID}", "${env.JOB_NAME}", output)
        }
    }
}
