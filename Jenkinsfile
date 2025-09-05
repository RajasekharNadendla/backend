pipeline{
    agent {
        label 'AGENT-1'
    }
    options{
        timeout(time: 30 , unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    environment{
        def appVersion = ''
    }
    stages{
         stage('Read the version'){
            steps{
                script{
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "application version is : $appVersion"
                }
            }
        }
        stage('Install npm Dependencies'){
            steps{
                sh """
                    npm install
                    ls -ltr
                    echo "application version is : $appVersion"
                """
            }
        }
        
    }
    post{
        always{
            echo 'I will run always'
            deleteDir()
        }
        failure { 
            echo 'I will run when the build failed!'
        }
        success { 
            echo 'I will run when build is success!'
        }
    }

}