pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"

        //input 'Build with maven?'

        sh '''
                    env
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                    echo "Build Number = ${BUILD_ID}"
                    mvn -B clean install -Dmaven.test.skip=true -U
                '''
        //input 'Continue?'
      }
    }
    stage('Scan App - Build Container') {
      parallel {
        stage('IQ-BOM') {
          steps {
            //input 'Scan with IQ at Build?'

            nexusPolicyEvaluation(iqApplication: 'WebGoat', iqStage: 'build', iqScanPatterns: [[scanPattern: '']])
          }
        }
        stage('Static Analysis') {
          steps {
            echo '...run SonarQube or other SAST tools here'
          }
        }
        stage('Build Container') {
          steps {
    
            //input 'Build with docker?'

            sh '''cd webgoat-server
                  /usr/local/bin/docker build -t webgoat/webgoat-8.0-${BUILD_ID} .
                    '''
          }
        }
      }
    }
    stage('Test Container') {
      parallel {
        stage('Test Container') {
          steps {
            catchError() {
              echo 'Test Container here'
            }

          }
        }
        stage('IQ-Scan Container') {
          steps {
            //input 'Scan with IQ at Stage-Release?'

            sh '/usr/local/bin/docker save webgoat/webgoat-8.0-${BUILD_ID} -o $WORKSPACE/webgoat.tar'
            nexusPolicyEvaluation(iqStage: 'stage-release', iqApplication: 'WebGoat')
          }
        }
      }
    }
    stage('Publish Container') {
      when {
        branch 'develop'
      }
      steps {
        //input 'Push to Nexus Repo?'

        sh '''
                    /usr/local/bin/docker tag webgoat/webgoat-8.0-${BUILD_ID} localhost:18443/webgoat/webgoat-8.0-${BUILD_ID}:8.0-${BUILD_ID}
                    /usr/local/bin/docker push localhost:18443/webgoat/webgoat-8.0-${BUILD_ID}
                '''
      }
    }
    stage('Create Tag') {
      when {
        branch 'develop'
      }
      steps {
        //input 'Create tag in Nexus Repo?'

        createTag nexusInstanceId: 'nexus', tagName: "DockerStagingDemoJenkinsfile-Webgoat8-${env.BUILD_ID}"

        associateTag nexusInstanceId: 'nexus', search: [[key: 'repository', value: 'docker-hosted-beta'], [key: 'name', value: "webgoat/webgoat-8.0-${env.BUILD_ID}"], [key: 'version', value: "8.0-${env.BUILD_ID}"]], tagName: "DockerStagingDemoJenkinsfile-Webgoat8-${env.BUILD_ID}"

        input 'Move Image out of Beta?'

        moveComponents destination: 'docker-hosted', nexusInstanceId: 'nexus', tagName: "DockerStagingDemoJenkinsfile-Webgoat8-${env.BUILD_ID}"



      }
    }
  }
  tools {
    maven 'localmaven'
  }
}
