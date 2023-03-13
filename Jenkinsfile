// week5 example uses Jenkin's "scripted" syntax, as opposed to its "declarative" syntax
// see: https://www.jenkins.io/doc/book/pipeline/syntax/#scripted-pipeline

// Defines a Kubernetes pod template that can be used to create nodes.

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
            git 'https://github.com/evan-jk/Continuous-Delivery-with-Docker-and-Jenkins-Second-Edition.git'
            container('gradle') {
              stage('Build a gradle project') {
                sh '''
                pwd
                cd Chapter08/sample1
                chmod +x gradlew
                ./gradlew build
                find . -name "*.jar"
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

                        // from the HTML publisher plugin
                        // https://www.jenkins.io/doc/pipeline/steps/htmlpublisher/
                        publishHTML(target: [
                            reportDir: 'Chapter08/sample1/build/reports/tests/test',
                            reportFiles: 'index.html',
                            reportName: 'JaCoCo Report'
                        ])
                    }
                }

                stage('Checkstyle Test') {
                    if (env.BRANCH_NAME != 'main') {
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

                        // from the HTML publisher plugin
                        // https://www.jenkins.io/doc/pipeline/steps/htmlpublisher/
                        publishHTML(target: [
                        reportDir: 'Chapter08/sample1/build/reports/checkstyle',
                        reportFiles: 'main.html',
                        reportName: 'Jacoco Checkstyle'
                    ])
                    }
                }
            }
        }

        stage('Build Java Image') {
            container('kaniko') {
                stage('Build a gradle project') {
                  sh '''
                  echo 'FROM openjdk:8-jre' > Dockerfile
                  echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
                  echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
                  find . -name "*.jar"
                  mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
                  /kaniko/executor --context `pwd` --destination evanjk/hello-kaniko:1.0
                  '''
                }
            }
        }
        
    }
}
