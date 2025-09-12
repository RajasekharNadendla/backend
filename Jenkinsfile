pipeline{
    agent {
        label 'AGENT-1'
    }
    options{
        timeout(time: 30 , unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    parameters{
        booleanParam(name: 'deploy', defaultValue: false, description: "Parameter to deploy")
    }
    environment{
        def appVersion = ''
        nexusUrl= 'nexus.rajasekhar.store:8081'
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
        stage('Build'){
            steps{
                sh """
                    zip -r -q backend-${appVersion}.zip * -x Jenkinsfile -x backend-${appVersion}.zip
                    ls -ltr
                """
            }
        }
        stage('sonar scan'){
            environment{
                scannerHome = tool 'sonar-6.0'
            }
            steps{
                script{
                    withSonarQubeEnv('sonar-6.0'){
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                    
                }
            }
        }
        stage('Quality Gate'){
            steps{
                timeout(time: 30, unit: 'MINUTES'){
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Nexus Artifact Upload'){
            steps{
                script{
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${nexusUrl}",
                        groupId: 'com.expense',
                        version: "${appVersion}",
                        repository: "backend",
                        credentialsId: 'nexus-auth',
                        artifacts: [
                        [artifactId: "backend",
                            classifier: '',
                            file: "backend-" + "${appVersion}" + '.zip',
                            type: 'zip']
                        ]
                    )

                }
            }
        }
        stage('Deploy'){
            when{
                expression{
                    params.deploy
                }
            }
            steps {
                 script{
                    def params = [
                        string(name: 'appVersion', value: "${appVersion}")
                    ]
                    build job: 'backend-deploy', parameters: params, wait: false
                }
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