pipeline {
    agent {
        label 'jenkins-slave'
    }
    tools {
        nodejs 'NodeJS-12'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Archive') {
            when {
                branch 'master'
            }
            stages {
                stage('minimize src') {
                    parallel {
                        stage('css') {
                            steps {
                                sh '''
                                    cd $WORKSPACE/www/css
                                    cleancss -d style.css > ../min/custom-min.
                                '''
                            }
                        }
                        stage('js') {
                            steps {
                                sh '''
                                    cd $WORKSPACE/www/js
                                    uglifyjs --timings init.js -o ../min/custom-min.js
                                '''
                            }
                        }
                    }
                }
                stage('Create tar') {
                    steps {
                        sh '''
                            cd ${WORKSPACE}/www   BASE_VERSION-BUILD_NUMBER
                            tar --exclude='./css' --exclude='./js' -c -z -f ../archive.tgz .
                        '''
                    }
                }
                stage('upload_to_nexus') {
                    steps {
                        script {
                            def baseVersion = readFile 'version.txt'
                            nexusArtifactUploader (
                                nexusUrl: '54.93.230.9:8081',
                                nexusVersion: 'nexus3', protocol: 'https',
                                credentialsId: 'nexus-key',
                                groupId: 'site', 
                                repository: 'raw-mdt-lab',
                                version: "${baseVersion}-${BUILD_NUMBER}",
                                artifacts: [
                                    [artifactId: 'archive', classifier: "${baseVersion}-${BUILD_NUMBER}", file: 'archive.tgz', type: 'tgz']
                                ]
                            )
                        }
                    }
                }
            }
            post {
                success {
                    writeFile file: '../deploy_version', text: "${baseVersion}-${BUILD_NUMBER}"
                }
            }
        }
    }
    post {
        always {
            sh 'git clean -fdx'
        }
    }
}