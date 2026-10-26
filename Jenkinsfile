pipeline {
  agent any

 environment {
    // Nexus creds: creates $NEXUS_CREDS, $NEXUS_CREDS_USR, $NEXUS_CREDS_PSW
    NEXUS_CREDS = credentials('nexus-creds')

    // GAV + Nexus repos
    GROUP_ID       = 'com.example'
    ARTIFACT_ID    = 'hello-world'
    VERSION        = '1.0.0-SNAPSHOT'
    NEXUS_BASE     = 'https://nexus.blackcloud-ug.eu'
  }

  stages {
    stage('Prep Java & Maven') {
      steps {
        sh '''
          java -version
          mvn -version
        '''
      }
    }

    stage('Write Maven settings.xml (safe)') {
  steps {
    sh '''
      cat > settings.xml.template <<'XML'
<settings>
  <servers>
    <server>
      <id>nexus-releases</id>
      <username>{{NEXUS_USER}}</username>
      <password>{{NEXUS_PASS}}</password>
    </server>
    <server>
      <id>nexus-snapshots</id>
      <username>{{NEXUS_USER}}</username>
      <password>{{NEXUS_PASS}}</password>
    </server>
    <server>
      <id>nexus-public</id>         
      <username>{{NEXUS_USER}}</username>
      <password>{{NEXUS_PASS}}</password>
    </server>
  </servers>
  <mirrors>
    <mirror>
      <id>nexus-public</id>
      <mirrorOf>*</mirrorOf>
      <url>{{NEXUS_BASE}}/repository/maven-public/</url>
    </mirror>
  </mirrors>
</settings>
XML
      sed -i "s|{{NEXUS_USER}}|${NEXUS_CREDS_USR}|g; s|{{NEXUS_PASS}}|${NEXUS_CREDS_PSW}|g; s|{{NEXUS_BASE}}|${NEXUS_BASE}|g" settings.xml.template
      mv settings.xml.template settings.xml
      chmod 600 settings.xml
    '''
  }
}   

    stage('Build & Publish (deploy)') {
      steps {
        sh '''
          mvn -s settings.xml -DskipTests clean deploy
        '''
      }
    }

    stage('Fetch from Nexus (resolve SNAPSHOT via Maven)') {
      steps {
        sh '''
          mvn -s settings.xml -q org.apache.maven.plugins:maven-dependency-plugin:3.6.1:copy \
             -Dartifact="$GROUP_ID:$ARTIFACT_ID:$VERSION:jar" \
             -DoutputDirectory=. \
             -DoverWrite=true
          ls
          # Rename the downloaded jar for the next stage
          mv ${ARTIFACT_ID}-*.jar app.jar
          ls -lh app.jar
        '''
      }
    }

    stage('Run app (simple)') {
      steps {
        sh '''
          java -jar app.jar
        '''
      }
    }
  }

  post {
    always {
      sh 'shred -u settings.xml 2>/dev/null || rm -f settings.xml' // we can delete the settings.xml to hide user sensetive data. shred -u = overwrite the file with random data before deleting.
      archiveArtifacts artifacts: 'app.jar, app.log', onlyIfSuccessful: false
    }
  }
}
