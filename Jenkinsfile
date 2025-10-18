pipeline {
  agent any

  tools { maven 'M2_HOME'; jdk 'JDK17' }

  environment {
    // --- Application info ---
    APP_NAME    = 'country-service'
    APP_VERSION = '1.0.0'
    GROUP_ID    = 'com.example'

    // --- Git ---
    GIT_BRANCH  = 'main'
    GIT_URL     = 'https://github.com/nadabouaouaja/compte-service-1.git'

    // --- SonarQube ---
    SONARQUBE_NAME = 'MySonarQubeServer'

    // --- Nexus repository ---
    NEXUS_URL  = 'http://localhost:8091/repository/maven-releases/'
    NEXUS_USER = 'admin'
    NEXUS_PASS = '197123'
    NEXUS_REPO_ID = 'nexus'

    // --- Tomcat Manager ---
    TOMCAT_URL  = 'http://jenkins:jenkins123@localhost:8090/manager/text'
  }

  stages {

    stage('Checkout') {
      steps {
        git branch: env.GIT_BRANCH, url: env.GIT_URL
      }
    }

    stage('Build & Test (H2)') {
      steps {
        sh '''
          set -e
          echo "Building and testing with H2 in-memory database..."

          mvn -B -U clean test \
            -Dspring.datasource.url="jdbc:h2:mem:testdb;MODE=MySQL;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE" \
            -Dspring.datasource.driver-class-name=org.h2.Driver \
            -Dspring.datasource.username=sa \
            -Dspring.datasource.password= \
            -Dspring.jpa.hibernate.ddl-auto=update \
            -Dspring.jpa.database-platform=org.hibernate.dialect.MySQLDialect \
            -Dspring.sql.init.mode=never

          echo "Tests passed - packaging application..."
          mvn -B -U -DskipTests package
        '''
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
            mvn -B -U -DskipTests sonar:sonar \
              -Dsonar.projectKey=${APP_NAME} \
              -Dsonar.projectName=${APP_NAME} \
              -Dsonar.projectVersion=${APP_VERSION}
          """
        }
      }
    }

    stage('Upload to Nexus') {
      steps {
        sh '''
          echo "Uploading artifact to Nexus..."
          ART=$(ls target/*.jar 2>/dev/null || ls target/*.war 2>/dev/null)
          if [ -z "$ART" ]; then echo "No artifact found in target/"; exit 1; fi

          mvn -B -U deploy:deploy-file \
            -Durl=${NEXUS_URL} \
            -DrepositoryId=${NEXUS_REPO_ID} \
            -Dfile="$ART" \
            -DgroupId=${GROUP_ID} \
            -DartifactId=${APP_NAME} \
            -Dversion=${APP_VERSION} \
            -Dpackaging=${ART##*.} \
            -DgeneratePom=true \
            -DretryFailedDeploymentCount=3 \
            -Dusername=${NEXUS_USER} \
            -Dpassword=${NEXUS_PASS}
        '''
      }
    }

    stage('Deploy to Tomcat') {
      when { branch 'main' }
      steps {
        sh '''
          echo "Deploying to Tomcat..."
          ART=$(ls target/*.war 2>/dev/null || true)
          if [ -z "$ART" ]; then echo "No WAR found, check Maven packaging"; exit 1; fi

          curl -sS -f -T "$ART" "${TOMCAT_URL}/deploy?path=/${APP_NAME}&update=true"
          echo "Deployment completed on Tomcat."
        '''
      }
    }
  }

  post {
    success { echo 'Pipeline completed successfully: Build + Test + Sonar + Nexus + Tomcat.' }
    failure { echo 'Pipeline failed - check Jenkins logs.' }
  }
}
