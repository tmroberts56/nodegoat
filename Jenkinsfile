pipeline {
    agent any

    environment {
        VERACODE_APP_NAME = 'NodeGoat'      // App Name in the Veracode Platform
    }

    tools {
       //'org.jenkinsci.plugins.docker.commons.tools.DockerTool' '18.09' 
       docker 'docker-latest'
    }

    stages{
        stage ('environment verify') {
            steps {
                sh 'pwd'
                sh 'ls -la'
            }
        }

        stage ('build') {
            steps {
                // use the NodeJS plugin
                nodejs(nodeJSInstallationName: 'NodeJS-12.0.0') {
                    sh 'npm config ls'
                    sh 'npm --version'
                    sh 'npm install'
                }
            }
        }

        stage ('Veracode scan') {
            steps {
                // zip archive for Veracode scanning.  Only include stuff we need,
                //  aka skip things like node_modules directory
                zip zipFile: 'upload.zip', archive: false, glob: '*.js,*.json,app/**,artifacts/**,config/**'

                echo 'Veracode scanning'
                withCredentials([ usernamePassword ( 
                    credentialsId: 'veracode_login', usernameVariable: 'VERACODE_API_ID', passwordVariable: 'VERACODE_API_KEY') ]) {
                        // fire-and-forget 
                        veracode applicationName: "${VERACODE_APP_NAME}", criticality: 'VeryHigh', debug: true, fileNamePattern: '', pHost: '', pPassword: '', pUser: '', replacementPattern: '', sandboxName: '', scanExcludesPattern: '', scanIncludesPattern: '', scanName: "${BUILD_TAG}", uploadExcludesPattern: '', uploadIncludesPattern: 'upload.zip', useIDkey: true, vid: "${VERACODE_API_ID}", vkey: "${VERACODE_API_KEY}"

                        // wait for scan to complete (timeout: x)
                        //veracode applicationName: "${VERACODE_APP_NAME}", criticality: 'VeryHigh', debug: true, timeout: 20, fileNamePattern: '', pHost: '', pPassword: '', pUser: '', replacementPattern: '', sandboxName: '', scanExcludesPattern: '', scanIncludesPattern: '', scanName: "${BUILD_TAG}", uploadExcludesPattern: '', uploadIncludesPattern: 'upload.zip', useIDkey: true, vid: "${VERACODE_API_ID}", vkey: "${VERACODE_API_KEY}"
                    }      
            }
        }

        /* srcclr can't handle a file:// as a git remote, so we need to set the git origin
            to the GitLab URL for this to work
        */
        stage ('Veracode SCA') {
            steps {
                echo 'Veracode SCA'
                withCredentials([ string(credentialsId: 'SCA_Token', variable: 'SRCCLR_API_TOKEN')]) {
                    nodejs(nodeJSInstallationName: 'NodeJS-12.0.0') {
                        sh 'git remote show origin'
                        //sh 'git remote remove origin'
                        //sh 'git remote add origin https://gitlab.com/veracode-demo-labs/nodegoat.git'
                        //sh 'git remote show origin'
                        //sh "curl -sSL https://download.sourceclear.com/ci.sh | sh"


                        //sh "curl -sSL https://download.sourceclear.com/ci.sh | DEBUG=1 sh -s -- scan --no-upload"
                        //sh "curl -sSL https://download.sourceclear.com/ci.sh | sh -s -- scan --no-upload"
                        
                        //sh "curl -sSL https://download.sourceclear.com/ci.sh | SRCCLR_SCM_URI=https://gitlab.com/veracode-demo-labs/nodegoat.git sh"
                    }
                }
            }
        }

        stage ('Docker-ize') {
            when { expression { isUnix() == true } }
            steps {
                echo 'Docker-izing'

                sh 'docker version'
                //script {
                //    def myImage = docker.build('nodegoat:snapshot')
               // }
            }
        }

        stage ('Deploy') {
            when { expression { isUnix() == true } }
            steps {
                echo 'Deploying to Heroku'
            }
        }
    }
}
