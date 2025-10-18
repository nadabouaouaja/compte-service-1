pipeline {
  agent any
  tools { maven 'mymaven'; jdk 'jdk17' }

  environment {
    NEXUS_URL = "http://localhost:8091/repository/maven-releases/"
    NEXUS_USER = "admin"
    NEXUS_PASS = "197123"

    TOMCAT_URL = "http://admin:admin123@localhost:8090/manager/text"

    SONARQUBE_NAME = "MySonarQubeServer"   // ex: MySonarQube (Configure System)
    APP_NAME = "country-service"
    APP_VERSION = "1.0.0"
    GROUP_ID = "com.example"
  }

  stages {

    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/nadabouaouaja/compte-service-1.git'
      }
    }

    stage('Build & Test') {
      steps {
        sh 'mvn -B clean package'
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
        }
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv(env.SONARQUBE_NAME) {
          sh """
            mvn -B sonar:sonar \
              -Dsonar.projectKey=${APP_NAME} \
              -Dsonar.projectName=${APP_NAME} \
              -Dsonar.projectVersion=${APP_VERSION}
          """
        }
      }
    }

    stage('Upload to Nexus') {
      steps {
        sh """
          mvn -B deploy:deploy-file \
            -Durl=${NEXUS_URL} -DrepositoryId=nexus \
            -Dfile=\$(ls target/*.war | head -1) \
            -DgroupId=${GROUP_ID} -DartifactId=${APP_NAME} -Dversion=${APP_VERSION} \
            -Dpackaging=war -DgeneratePom=true -DretryFailedDeploymentCount=3 \
            -Dusername=${NEXUS_USER} -Dpassword=${NEXUS_PASS}
        """
      }
    }

    stage('Deploy to Tomcat') {
      steps {
        sh """
          WAR=\$(ls target/*.war | head -1)
          curl -T "$WAR" "${TOMCAT_URL}/deploy?path=/${APP_NAME}&update=true"
        """
      }
    }

  }

  post {
    success { echo '✅ CI/CD terminé : Build, Tests, Sonar (sans Quality Gate), Nexus, Tomcat.' }
    failure { echo '❌ Échec — vois les logs de stages.' }
  }
}
