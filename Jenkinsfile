podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: gradle
        image: gradle:6.3-jdk14
        command:
        - sleep
        args:
        - 99d
        volumeMounts:
        - name: shared-storage
          mountPath: /mnt
      - name: kaniko
        image: gcr.io/kaniko-project/executor:debug
        command:
        - sleep
        args:
        - 9999999
        volumeMounts:
        - name: shared-storage
          mountPath: /mnt
        - name: kaniko-secret
          mountPath: /kaniko/.docker
      restartPolicy: Never
      volumes:
      - name: shared-storage
        persistentVolumeClaim:
          claimName: jenkins-pv-claim
      - name: kaniko-secret
        secret:
            secretName: dockercred
            items:
            - key: .dockerconfigjson
              path: config.json
''') {
  node(POD_LABEL) {
    stage('Build a gradle project') {
      container('gradle') {
        git 'https://github.com/evan-jk/Continuous-Delivery-with-Docker-and-Jenkins-Second-Edition.git'
        stage('Build a gradle project') {
          sh '''
          cd Chapter08/sample1
          chmod +x gradlew
          ./gradlew build
          mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt
          '''
        }

        stage('Code coverage') {
          if (env.BRANCH_NAME == 'main') {
            echo "I am the ${env.BRANCH_NAME} branch"
            try {
              sh '''
              pwd
              cd Chapter08/sample1
              ./gradlew jacocoTestCoverageVerification
              ./gradlew jacocoTestReport
              '''
            } catch (Exception E) {
              echo 'Failure detected'
            }

          }
        }

        stage('Checkstyle Test') {
          if (env.BRANCH_NAME == 'feature' || env.BRANCH_NAME == 'main') {
            echo "I am the ${env.BRANCH_NAME} branch"
            try {
              sh '''
              pwd
              cd Chapter08/sample1
              ./gradlew checkstyleMain
              '''
            } catch (Exception E) {
              echo 'Failure detected'
            }
          } else {
            echo 'this is the playground branch - no testing'
          }
        }
      }
    }

    stage('Build Java Image') {
      if ( "${currentBuild.currentResult}" == "SUCCESS" && env.BRANCH_NAME != 'playground') {
        container('kaniko') {
        stage('Kaniko stage') {
              stage('Build a gradle project') {
              sh '''
                echo 'FROM openjdk:8-jre' > Dockerfile
                echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
                echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
                mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
                /kaniko/executor --context `pwd` --destination evanjk/hello-test.1
                '''
              }
        }
      }
      }
      
    }
  }
}
