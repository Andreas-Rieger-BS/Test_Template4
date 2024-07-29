pipeline {
    agent any 
    environment {
        GLOBAL_AGENT_HTTP_PROXY = credentials('http-proxy')
        GLOBAL_AGENT_HTTPS_PROXY = credentials('https-proxy')
        GITHUB_AUTH_TOKEN = credentials('github-token')
        GITHUB_MAIL = credentials('github-mail')
    }
    stages {
        stage('Build') {
            steps {
                echo 'Hello world!' 
            }
        }
        stage('SonarQube analysis') {
            steps {
                script {
                    scannerHome = tool 'sonarqubeScannerInstallation'// must match the name of an actual scanner installation directory on your Jenkins build agent
                }
                withSonarQubeEnv('sonarqubeInstallation') {// If you have configured more than one global server connection, you can specify its name as configured in Jenkins
                  sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
        stage('bomBuilder') {
            steps {
                sh """                
                    git config --global http.proxy $GLOBAL_AGENT_HTTP_PROXY
                    git config --global https.proxy $GLOBAL_AGENT_HTTPS_PROXY
                    git config --global user.email $GITHUB_MAIL
                    git config --global user.name "Andreas Rieger"
                    cd ..
                    git clone https://github.com/Andreas-Rieger-BS/Test_Template4.git
                    /cyclonedx-linux-x64 add files --no-input --output-format xml --exclude /.git/** --exclude cyclonedx-linux-x64 --output-file bom.xml --base-path Test_Template4/
                    cp bom.xml Test_Template4_main/
                    mv bom.xml Test_Template4/
                    cd Test_Template4
                    git remote set-url origin https://$GITHUB_AUTH_TOKEN@github.com/Andreas-Rieger-BS/Test_Template4.git
                    git add .
                    git commit -m "add bom.xml file"
                    git push origin
                    cd /var/lib/jenkins/workspace/Test_Template4_main
                """
            }
        }
        stage('dependencyTrackPublisher') {
            steps {
                 dependencyTrackPublisher artifact: 'bom.xml', projectName: 'Test_Template4', projectVersion: '1.0', synchronous: true
            }
        }
    }
}
