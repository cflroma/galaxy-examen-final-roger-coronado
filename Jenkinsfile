pipeline {

  agent any

  stages {
    stage('Checkout repository') {
      steps {
        cleanWs()
        checkout scm
      }
    }

    stage('Build') {
      agent {
        docker {
          image 'maven:3.6.3-openjdk-11-slim'
        }
      }
      steps {
        script {
          sh 'java -version'
          sh 'mvn -version'
          sh 'mvn verify'
          sh 'mvn clean package'
          sh 'ls -l target/'
        }
      }
      post {
        success {
          archiveArtifacts artifacts: 'target/labmaven-*.jar', fingerprint: true, onlyIfSuccessful: true
        }
      }
    }

    stage('SonarQube') {
      agent {
        docker {
          image 'maven:3.6.3-openjdk-11-slim'
        }
      }
      steps {
        script {
          def scannerHome = tool 'scanner-default'
          withSonarQubeEnv('sonar-server') {
            sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=examen01 \
                            -Dsonar.projectName=examen01 \
                            -Dsonar.sources=src/main \
                            -Dsonar.sourceEncoding=UTF-8 \
                            -Dsonar.language=java \
                            -Dsonar.tests=src/test \
                            -Dsonar.junit.reportsPath=target/surefire-reports \
                            -Dsonar.surefire.reportsPath=target/surefire-reports \
                            -Dsonar.jacoco.reportPath=target/jacoco.exec \
                            -Dsonar.java.binaries=target/classes \
                            -Dsonar.java.coveragePlugin=jacoco \
                            -Dsonar.coverage.jacoco.xmlReportPaths=target/jacoco.xml \
                            -Dsonar.exclusions=**/*IT.java,**/*TEST.java,**/*Test.java,**/src/it**,**/src/test**,**/gradle/wrapper** \
                            -Dsonar.java.libraries=target/*.jar"
          }
        }
      }
    }

    stage('Build Image') {
      steps {
        copyArtifacts filter: 'target/*.jar',
                      fingerprintArtifacts: true,
                      projectName: '${JOB_NAME}',
                      flatten: true,
                      selector: specific("${BUILD_NUMBER}"),
                      target: 'target'
        sh 'docker --version'
        sh 'docker-compose --version'
        sh 'docker-compose build'
      }
    }

    stage('Publish Image') {
      environment {
        DOCKER_CREDS = credentials('docker-credentials')
      }
      steps {
        script {
          sh 'docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}'
          sh 'docker tag msmicroservice ${DOCKER_CREDS_USR}/msmicroservice:${BUILD_NUMBER}'
          sh 'docker push ${DOCKER_CREDS_USR}/msmicroservice:${BUILD_NUMBER}'
          sh 'docker logout'
        }
      }
    }

    stage('Run Container') {
      environment {
        DOCKER_CREDS = credentials('docker-credentials')
      }
      steps {
        script {
          sh 'docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}'
          sh 'docker rm -f msmicroservice'
        //  sh 'docker run -d --name msmicroservice -p 8080:8080 ${DOCKER_CREDS_USR}/msmicroservice:${BUILD_NUMBER}'
          sh 'docker logout'
        }
      }
    }

  }

}
