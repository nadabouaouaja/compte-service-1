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

  options { timestamps(); ansiColor('xterm') }

  environment {
    // A ADAPTER
    GIT_URL            = 'https://github.com/nadabouaouaja/compte-service-1.git'
    SONARQUBE_NAME     = 'MySonarQubeServer'   // nom configuré côté Jenkins
    NEXUS_RELEASES_URL = 'http://localhost:8091/repository/maven-releases/'
    NEXUS_SNAPSHOTS_URL= 'http://localhost:8091/repository/maven-snapshots/'
    NEXUS_CRED_ID      = 'nexus-creds'         // Credentials Jenkins (Username with password)
    TOMCAT_URL_BASE    = 'http://localhost:8090/manager/text'
    TOMCAT_CRED_ID     = 'tomcat-creds'        // Credentials Jenkins (Username with password)
    TOMCAT_ENABLED     = 'false'               // passe à true si tu veux déployer
    // Fin A ADAPTER
  }

  tools { maven 'M2_HOME'; jdk 'JDK17' }

  stages {

    stage('Checkout') {
      steps {
        git branch: 'main', url: env.GIT_URL
      }
    }

    stage('Build & Tests (H2)') {
      steps {
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

          mvn -B -U -DskipTests package
        '''
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
          archiveArtifacts allowEmptyArchive: true, artifacts: 'target/*.jar, target/*.war'
        }
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv(env.SONARQUBE_NAME) {
          sh 'mvn -B -U -DskipTests sonar:sonar'
        }
      }
    }

    stage('Upload to Nexus (auto SNAPSHOT/RELEASE)') {
      steps {
        withCredentials([usernamePassword(credentialsId: env.NEXUS_CRED_ID, usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
          sh '''
            set -e

            echo "[NEXUS] Lecture de la version et du packaging depuis le POM…"
            VERSION=$(mvn -q -Dexec.executable=echo -Dexec.args='${project.version}' --non-recursive exec:exec)
            PACKAGING=$(mvn -q -Dexec.executable=echo -Dexec.args='${project.packaging}' --non-recursive exec:exec)

            echo "[NEXUS] Version détectée: $VERSION"
            echo "[NEXUS] Packaging détecté: $PACKAGING"

            echo "[NEXUS] Recherche de l’artefact construit dans target/…"
            ART=$(ls target/*.${PACKAGING} 2>/dev/null || true)
            if [ -z "$ART" ]; then
              echo "ERREUR: Aucun artefact *.${PACKAGING} dans target/"
              echo "Contenu de target/:"
              ls -la target/ || true
              exit 2
            fi
            echo "[NEXUS] Artefact: $ART"

            if echo "$VERSION" | grep -qi SNAPSHOT; then
              REPO_URL="${NEXUS_SNAPSHOTS_URL}"
              echo "[NEXUS] Version SNAPSHOT -> dépôt: ${REPO_URL}"
            else
              REPO_URL="${NEXUS_RELEASES_URL}"
              echo "[NEXUS] Version RELEASE -> dépôt: ${REPO_URL}"
            fi

            echo "[NEXUS] Création d’un settings.xml temporaire…"
            cat > settings-nexus.xml <<EOF
            <settings>
              <servers>
                <server>
                  <id>nexus</id>
                  <username>${NEXUS_USER}</username>
                  <password>${NEXUS_PASS}</password>
                </server>
              </servers>
            </settings>
            EOF

            GROUP_ID=$(mvn -q -Dexec.executable=echo -Dexec.args='${project.groupId}' --non-recursive exec:exec)
            ARTIFACT_ID=$(mvn -q -Dexec.executable=echo -Dexec.args='${project.artifactId}' --non-recursive exec:exec)

            echo "[NEXUS] Déploiement: ${GROUP_ID}:${ARTIFACT_ID}:${VERSION} (${PACKAGING}) -> ${REPO_URL}"
            mvn -s settings-nexus.xml -B -U deploy:deploy-file \
              -Durl="${REPO_URL}" \
              -DrepositoryId=nexus \
              -Dfile="${ART}" \
              -DgroupId="${GROUP_ID}" \
              -DartifactId="${ARTIFACT_ID}" \
              -Dversion="${VERSION}" \
              -Dpackaging="${PACKAGING}" \
              -DgeneratePom=true \
              -DretryFailedDeploymentCount=3

            echo "[NEXUS] Déploiement terminé."
          '''
        }
      }
    }

    stage('Deploy to Tomcat (si WAR)') {
      when { expression { return env.TOMCAT_ENABLED == 'true' } }
      steps {
        withCredentials([usernamePassword(credentialsId: env.TOMCAT_CRED_ID, usernameVariable: 'TUSER', passwordVariable: 'TPASS')]) {
          sh '''
            set -e

            VERSION=$(mvn -q -Dexec.executable=echo -Dexec.args='${project.version}' --non-recursive exec:exec)
            PACKAGING=$(mvn -q -Dexec.executable=echo -Dexec.args='${project.packaging}' --non-recursive exec:exec)
            ARTIFACT_ID=$(mvn -q -Dexec.executable=echo -Dexec.args='${project.artifactId}' --non-recursive exec:exec)

            if [ "$PACKAGING" != "war" ]; then
              echo "[TOMCAT] Packaging=$PACKAGING -> rien à déployer sur Tomcat. Étape ignorée."
              exit 0
            fi

            WAR=$(ls target/*.war | head -1)
            if [ -z "$WAR" ]; then
              echo "[TOMCAT] Aucun WAR trouvé dans target/"
              exit 2
            fi

            echo "[TOMCAT] Déploiement de $WAR sur ${TOMCAT_URL_BASE} (context=/${ARTIFACT_ID})"
            curl -sf -u "${TUSER}:${TPASS}" -T "$WAR" \
              "${TOMCAT_URL_BASE}/deploy?path=/${ARTIFACT_ID}&update=true"

            echo "[TOMCAT] Vérification état des apps:"
            curl -sf -u "${TUSER}:${TPASS}" "${TOMCAT_URL_BASE}/list" | sed -n '1,20p'
          '''
        }
      }
    }
  }

  post {
    success { echo 'Pipeline OK' }
    failure { echo 'Pipeline failed - check Jenkins logs.' }
  }
}

