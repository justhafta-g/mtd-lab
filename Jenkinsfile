pipeline {
    
     agent {
        label 'docker'
        }

    // pipeline parameters
    parameters {
        // Release type parameter
        choice (
            name: 'RELEASE_TYPE',
            choices: ['DEVELOP', 'TEST', 'RELEASE'], 
            description: 'Release type selection (DEVELOP|TEST|RELEASE)')
        
        // Release version parameter
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
        // ansiColor('xterm')
    }

    // global tools
    tools {
        nodejs 'Node12'
    }

    // pipeline steps
    stages {
        // CHECKOUT git repository 
        stage('Checkout') {
            steps {
                git 'https://github.com/justhafta-g/mtd-lab'
            }
        }

        // BUILD: clean .css files and minimize .js files
        stage('Build') {
            parallel {
                // minimize .js files 
                stage('JS') {
                    steps {
                        sh label: 'minimize JS', script: """
                        cd ${WORKSPACE}/www/js
                        uglifyjs --timings init.js -o ../min/custom-min.js
                        """
                    }
                }

                // lint .css files
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


                // clean .css files 
                stage('CSS') {
                    steps {
                        sh label: 'minimize CSS', script: """
                        cd ${WORKSPACE}/www/css
                        cleancss -d style.css > ../min/custom-min.css"""
                    }
                }
                
                // tar artifact
                stage('Tar artifact') {
                    steps {
                        sh label: 'archive', script: """
                        cd ${WORKSPACE}/www
                        tar --exclude='./css' --exclude='./js' -c -z -f ../site-archive-${params.RELEASE_TYPE}-${params.RELEASE_VER}-${BUILD_NUMBER}.tgz ."""
                    }
                }
            }
        }
          stage('upload_to_nexus') {
                    steps {
                        script {
                        nexusArtifactUploader artifacts: [[artifactId: 'site-archive', classifier: '', file: 'site-archive-DEVELOP-0.1.1-21.tgz', type: 'tgz']], credentialsId: 'f8190dea-f270-442e-b06b-2c7b87f9d919', groupId: 'site', nexusUrl: 'server2.jenkins-practice.tk', nexusVersion: 'nexus3', protocol: 'http', repository: 'student5-repo', version: '${RELEASE_TYPE}-${RELEASE_VER}-${BUILD_NUMBER}'
                    }
                }
            }

    }
}
