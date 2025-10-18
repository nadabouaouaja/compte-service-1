// pipeline {
//   agent any

//   tools { maven 'M2_HOME'; jdk 'JDK17' }

//   environment {
//     // --- Application info ---
//     APP_NAME    = 'country-service'
//     APP_VERSION = '1.0.0'
//     GROUP_ID    = 'com.example'

//     // --- Git ---
//     GIT_BRANCH  = 'main'
//     GIT_URL     = 'https://github.com/nadabouaouaja/compte-service-1.git'

//     // --- SonarQube ---
//     SONARQUBE_NAME = 'MySonarQubeServer'

//     // --- Nexus repository ---
//     NEXUS_URL  = 'http://localhost:8091/repository/maven-releases/'
//     NEXUS_USER = 'admin'
//     NEXUS_PASS = '197123'
//     NEXUS_REPO_ID = 'nexus'

//     // --- Tomcat Manager ---
//     TOMCAT_URL  = 'http://jenkins:jenkins123@localhost:8090/manager/text'
//   }

//   stages {

//     stage('Checkout') {
//       steps {
//         git branch: env.GIT_BRANCH, url: env.GIT_URL
//       }
//     }

//     stage('Build & Test (H2)') {
//       steps {
//         sh '''
//           set -e
//           echo "Building and testing with H2 in-memory database..."

//           mvn -B -U clean test \
//             -Dspring.datasource.url="jdbc:h2:mem:testdb;MODE=MySQL;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE" \
//             -Dspring.datasource.driver-class-name=org.h2.Driver \
//             -Dspring.datasource.username=sa \
//             -Dspring.datasource.password= \
//             -Dspring.jpa.hibernate.ddl-auto=update \
//             -Dspring.jpa.database-platform=org.hibernate.dialect.MySQLDialect \
//             -Dspring.sql.init.mode=never

//           echo "Tests passed - packaging application..."
//           mvn -B -U -DskipTests package
//         '''
//       }
//       post {
//         always {
//           junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
//         }
//       }
//     }

//     stage('SonarQube Analysis') {
//       steps {
//         withSonarQubeEnv(env.SONARQUBE_NAME) {
//           sh """
//             mvn -B -U -DskipTests sonar:sonar \
//               -Dsonar.projectKey=${APP_NAME} \
//               -Dsonar.projectName=${APP_NAME} \
//               -Dsonar.projectVersion=${APP_VERSION}
//           """
//         }
//       }
//     }

//     stage('Upload to Nexus') {
//       steps {
//         sh '''
//           echo "Uploading artifact to Nexus..."
//           ART=$(ls target/*.jar 2>/dev/null || ls target/*.war 2>/dev/null)
//           if [ -z "$ART" ]; then echo "No artifact found in target/"; exit 1; fi

//           mvn -B -U deploy:deploy-file \
//             -Durl=${NEXUS_URL} \
//             -DrepositoryId=${NEXUS_REPO_ID} \
//             -Dfile="$ART" \
//             -DgroupId=${GROUP_ID} \
//             -DartifactId=${APP_NAME} \
//             -Dversion=${APP_VERSION} \
//             -Dpackaging=${ART##*.} \
//             -DgeneratePom=true \
//             -DretryFailedDeploymentCount=3 \
//             -Dusername=${NEXUS_USER} \
//             -Dpassword=${NEXUS_PASS}
//         '''
//       }
//     }

//     stage('Deploy to Tomcat') {
//       when { branch 'main' }
//       steps {
//         sh '''
//           echo "Deploying to Tomcat..."
//           ART=$(ls target/*.war 2>/dev/null || true)
//           if [ -z "$ART" ]; then echo "No WAR found, check Maven packaging"; exit 1; fi

//           curl -sS -f -T "$ART" "${TOMCAT_URL}/deploy?path=/${APP_NAME}&update=true"
//           echo "Deployment completed on Tomcat."
//         '''
//       }
//     }
//   }

//   post {
//     success { echo 'Pipeline completed successfully: Build + Test + Sonar + Nexus + Tomcat.' }
//     failure { echo 'Pipeline failed - check Jenkins logs.' }
//   }
// }





// pipeline {
//   agent any

//   tools { maven 'M2_HOME'; jdk 'JDK17' }

//   // options {
//   //   timestamps()
//   //   ansiColor('xterm')
//   // }

//   environment {
//     // URLs cibles
//     NEXUS_RELEASES_URL = "http://localhost:8091/repository/maven-releases/"
//     NEXUS_SNAPSHOTS_URL = "http://localhost:8091/repository/maven-snapshots/"

//     // Métadonnées d’artefact si on utilise deploy-file
//     GROUP_ID     = "com.example"
//     ARTIFACT_ID  = "country-service"
//     VERSION      = "1.0.0"
//     PACKAGING    = "jar"       // jar ou war selon votre projet

//     // Sonar
//     SONARQUBE_NAME = "MySonarQubeServer"
//   }

//   stages {

//     stage('Checkout') {
//       steps {
//         // Soit checkout scm, soit git(...)
//         checkout scm
//         // git branch: 'main', url: 'https://github.com/nadabouaouaja/compte-service-1.git'
//       }
//     }

//     stage('Build & Tests (H2 en mémoire)') {
//       steps {
//         sh '''
//           set -e
//           mvn -B -U clean test \
//             -Dspring.datasource.url="jdbc:h2:mem:testdb;MODE=MySQL;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE" \
//             -Dspring.datasource.driver-class-name=org.h2.Driver \
//             -Dspring.datasource.username=sa \
//             -Dspring.datasource.password= \
//             -Dspring.jpa.hibernate.ddl-auto=update \
//             -Dspring.jpa.database-platform=org.hibernate.dialect.MySQLDialect \
//             -Dspring.sql.init.mode=never

//           mvn -B -U -DskipTests package
//         '''
//       }
//       post {
//         always {
//           junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
//         }
//       }
//     }

//     stage('SonarQube Analysis') {
//       steps {
//         withSonarQubeEnv(env.SONARQUBE_NAME) {
//           sh 'mvn -B -U -DskipTests sonar:sonar'
//         }
//       }
//     }

//     stage('Upload to Nexus (releases)') {
//       steps {
//         // Credentials Jenkins de type Username/Password, id = "nexus-creds"
//         withCredentials([usernamePassword(credentialsId: 'nexus-creds', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
//           sh '''
//             set -e

//             echo "Recherche de l’artefact construit..."
//             ART=$(ls target/*.jar 2>/dev/null || true)
//             if [ -z "$ART" ]; then
//               ART=$(ls target/*.war 2>/dev/null || true)
//             fi
//             if [ -z "$ART" ]; then
//               echo "Aucun .jar/.war trouvé dans target/"
//               exit 2
//             fi

//             echo "Création d’un settings.xml temporaire pour Nexus"
//             cat > settings-nexus.xml <<EOF
//             <settings>
//               <servers>
//                 <server>
//                   <id>nexus</id>
//                   <username>${NEXUS_USER}</username>
//                   <password>${NEXUS_PASS}</password>
//                 </server>
//               </servers>
//             </settings>
//             EOF

//             echo "Upload vers le dépôt releases"
//             mvn -s settings-nexus.xml -B -U deploy:deploy-file \
//               -Durl=${NEXUS_RELEASES_URL} \
//               -DrepositoryId=nexus \
//               -Dfile="${ART}" \
//               -DgroupId=${GROUP_ID} \
//               -DartifactId=${ARTIFACT_ID} \
//               -Dversion=${VERSION} \
//               -Dpackaging=${PACKAGING} \
//               -DgeneratePom=true \
//               -DretryFailedDeploymentCount=3
//           '''
//         }
//       }
//     }

//     stage('Deploy to Tomcat') {
//       when { branch 'main' }
//       steps {
//         // Credentials Jenkins de type Username/Password, id = "tomcat-creds"
//         withCredentials([usernamePassword(credentialsId: 'tomcat-creds', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
//           sh '''
//             set -e

//             WAR=$(ls target/*.war 2>/dev/null || true)
//             if [ -z "$WAR" ]; then
//               echo "Aucun WAR trouvé, déploiement Tomcat ignoré (Tomcat Manager déploie des WAR)."
//               exit 0
//             fi

//             APP_NAME="${ARTIFACT_ID}"
//             TOMCAT_URL="http://localhost:8090/manager/text"

//             echo "Déploiement sur Tomcat..."
//             curl --fail -u "${TOMCAT_USER}:${TOMCAT_PASS}" -T "${WAR}" \
//               "${TOMCAT_URL}/deploy?path=/${APP_NAME}&update=true"
//           '''
//         }
//       }
//     }
//   }

//   post {
//     success {
//       echo 'Pipeline OK'
//     }
//     failure {
//       echo 'Pipeline failed - check Jenkins logs.'
//     }
//   }
// }








pipeline {
  agent any

  // Conserve uniquement timestamps ici (ansiColor sera appliqué autour des steps)
  options { timestamps() }

  tools {
    // Adapte aux noms de tes tools Jenkins, ou supprime ce bloc si tu utilises une image Docker déjà équipée
    // maven 'Maven-3'
    // jdk   'JDK-17'
  }

  environment {
    // ====== A ADAPTER ======
    // 1) SonarQube (nom tel que déclaré in "Manage Jenkins" > "Configure System")
    SONARQUBE_NAME     = 'MySonarQubeServer'

    // 2) Nexus releases repo (URL EXACTE du repository cible)
    NEXUS_RELEASES_URL = 'http://localhost:8091/repository/maven-releases/'

    // 3) Credentials IDs créés dans Jenkins (Manage Jenkins > Credentials)
    NEXUS_CRED_ID      = 'nexus-credentials'     // username/password Nexus
    TOMCAT_CRED_ID     = 'tomcat-credentials'    // username/password Tomcat Manager

    // 4) Coordonnées de l’appli
    APP_NAME    = 'country-service'
    APP_VERSION = '1.0.0'       // si contient "SNAPSHOT", la phase Nexus Releases est ignorée
    GROUP_ID    = 'com.example' // groupId utilisé pour publier dans Nexus

    // 5) Tomcat Manager URL (sans credentials)
    // Exemple: http://localhost:8090/manager/text
    TOMCAT_MANAGER_URL = 'http://localhost:8090/manager/text'
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build & Tests (H2 en mémoire)') {
      steps {
        ansiColor('xterm') {
          sh '''
            set -e
            mvn -B -U clean test \
              -Dspring.datasource.url="jdbc:h2:mem:testdb;MODE=MySQL;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE" \
              -Dspring.datasource.driver-class-name=org.h2.Driver \
              -Dspring.datasource.username=sa \
              -Dspring.datasource.password= \
              -Dspring.jpa.hibernate.ddl-auto=update \
              -Dspring.jpa.database-platform=org.hibernate.dialect.MySQLDialect \
              -Dspring.sql.init.mode=never

            # Package sans relancer les tests
            mvn -B -U -DskipTests package
          '''
        }
      }
      post {
        always {
          // Publie les rapports de tests (ne casse pas la build si vide)
          junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
        }
      }
    }

    stage('SonarQube Analysis') {
      // N’exécute cette étape que si SONARQUBE_NAME est défini
      when { expression { return env.SONARQUBE_NAME?.trim() } }
      steps {
        withSonarQubeEnv(env.SONARQUBE_NAME) {
          ansiColor('xterm') {
            sh """
              mvn -B -U -DskipTests sonar:sonar \
                -Dsonar.projectKey=${GROUP_ID}:${APP_NAME} \
                -Dsonar.projectName=${APP_NAME} \
                -Dsonar.projectVersion=${APP_VERSION}
            """
          }
        }
      }
    }

    stage('Upload to Nexus (releases)') {
      // On ne déploie dans "releases" que si la version ne contient pas SNAPSHOT
      when {
        allOf {
          expression { return env.APP_VERSION && !env.APP_VERSION.contains('SNAPSHOT') }
          expression { return env.NEXUS_RELEASES_URL?.trim() }
        }
      }
      steps {
        withCredentials([usernamePassword(credentialsId: env.NEXUS_CRED_ID, usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
          ansiColor('xterm') {
            sh '''
              set -e
              echo "Recherche de l’artefact construit..."
              ART=$(ls target/*.jar 2>/dev/null | head -1)
              if [ -z "$ART" ]; then
                echo "Aucun JAR trouvé dans target/. Vérifie le packaging du projet."
                exit 1
              fi
              echo "Artefact: $ART"

              echo "Création d’un settings.xml temporaire pour Nexus"
              cat > settings-nexus.xml <<EOF
<settings>
  <servers>
    <server>
      <id>nexus</id>
      <username>$NEXUS_USER</username>
      <password>$NEXUS_PASS</password>
    </server>
  </servers>
</settings>
EOF

              mvn -B -U -s settings-nexus.xml deploy:deploy-file \
                -Durl="${NEXUS_RELEASES_URL}" \
                -DrepositoryId=nexus \
                -Dfile="$ART" \
                -DgroupId="${GROUP_ID}" \
                -DartifactId="${APP_NAME}" \
                -Dversion="${APP_VERSION}" \
                -Dpackaging=jar \
                -DgeneratePom=true \
                -DretryFailedDeploymentCount=3
            '''
          }
        }
      }
    }

    stage('Deploy to Tomcat') {
      // On déploie uniquement si un WAR a été généré et si on est sur la branche main
      when {
        allOf {
          branch 'main'
          expression { return fileExists('target') && sh(script: 'ls target/*.war >/dev/null 2>&1; echo $?', returnStdout: true).trim() == '0' }
        }
      }
      steps {
        withCredentials([usernamePassword(credentialsId: env.TOMCAT_CRED_ID, usernameVariable: 'TC_USER', passwordVariable: 'TC_PASS')]) {
          ansiColor('xterm') {
            sh '''
              set -e
              WAR=$(ls target/*.war | head -1)
              echo "WAR détecté: $WAR"
              # Déploiement via Tomcat Manager (update=true pour remplacer si déjà présent)
              curl -sS -u "$TC_USER:$TC_PASS" -T "$WAR" \
                "${TOMCAT_MANAGER_URL}/deploy?path=/${APP_NAME}&update=true"
              echo
              echo "Déploiement demandé au Tomcat Manager."
            '''
          }
        }
      }
    }
  }

  post {
    success { echo 'Pipeline OK' }
    failure { echo 'Pipeline failed - check Jenkins logs.' }
  }
}
