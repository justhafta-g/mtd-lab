pipeline {
    
     agent {
        label 'docker'
        }

    
    parameters {
       
        choice (
            name: 'RELEASE_TYPE',
            choices: ['DEVELOP', 'TEST', 'RELEASE'], 
            description: 'Release type selection (DEVELOP|TEST|RELEASE)')
        
        
        string (
            name: 'RELEASE_VER', 
            defaultValue: '0.1.1', 
            description: 'Release version (number value, format x.x.x)', 
            trim: false)
    }
    
    options {
        buildDiscarder(logRotator(artifactNumToKeepStr: '3', artifactDaysToKeepStr: '5', daysToKeepStr: '5', numToKeepStr: '3'))
        timeout(time: 1, unit: 'HOURS')
        timestamps()
        
    }

    tools {
        nodejs 'Node12'
    }

    
    stages {
        
        stage('Checkout') {
            steps {
                git 'https://github.com/justhafta-g/mtd-lab'
            }
        }
        
    stage (' Test ') {
            
        when{
               branch 'PR-*'  
            } 
        environment {
        scannerHome = tool 'sonarscanner4'
          }   
    
            steps {
            
            withSonarQubeEnv('sonarqube-external') {
            sh "${scannerHome}/bin/sonar-scanner\
              -Dsonar.projectKey=student5-project \
              -Dsonar.sources=. \
              -Dsonar.host.url=http://server1.jenkins-practice.tk\
              -Dsonar.login=45cacf83fe2e56ed78b6c11f9fa1acb8461531cc"
                 }
        timeout(time: 3, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
                    }
            }

        }
        stage('Build') {
            parallel {
                
                stage('JS') {
                    steps {
                        sh label: 'minimize JS', script: """
                        cd ${WORKSPACE}/www/js
                        uglifyjs --timings init.js -o ../min/custom-min.js
                        """
                    }
                }

                
                stage('TEST') {
                    when {
                        allOf {
                            branch 'master'
                            expression {
                                params.RELEASE_TYPE == 'TEST'
                            }
                        }
                    }
                

                    steps {
                        sh label: 'lint CSS', script: """
                        cd ${WORKSPACE}/www/css
                        stylelint style.css"""
                    }
                }


                stage('CSS') {
                    steps {
                        sh label: 'minimize CSS', script: """
                        cd ${WORKSPACE}/www/css
                        cleancss -d style.css > ../min/custom-min.css"""
                    }
                }
                
                
                stage('Tar artifact') {
                    steps {
                        sh label: 'archive', script: """
                        cd ${WORKSPACE}/www
                        tar --exclude='./css' --exclude='./js' -c -z -f ../site-archive-${RELEASE_TYPE}-${RELEASE_VER}-${BUILD_NUMBER}.tgz ."""
                    }
                }
            }
        }
          stage('upload_to_nexus') {
                    steps {
                        script {
                        nexusArtifactUploader artifacts: [[artifactId: 'site-archive', classifier: '', file: "site-archive-${RELEASE_TYPE}-${RELEASE_VER}-${BUILD_NUMBER}.tgz", type: 'tgz']], credentialsId: 'f8190dea-f270-442e-b06b-2c7b87f9d919', groupId: 'site', nexusUrl: 'server2.jenkins-practice.tk', nexusVersion: 'nexus3', protocol: 'https', repository: 'student5-repo', version: '${RELEASE_TYPE}-${RELEASE_VER}-${BUILD_NUMBER}'
                    }
                }
            }

    }
}
