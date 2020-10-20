pipeline{
    agent { label 'master' }
    stages{
        stage('Running-Unit-Test'){
            steps{
                sh 'mvn test'
            }
            post{
                success{
                    step([$class: 'JUnitResultArchiver', testResults: 'target/surefire-reports/*.xml'])
                }
            }
        }
        stage('Running Build'){
            steps{
                sh 'mvn clean install'
            }
            post {
                always {
                    archiveArtifacts artifacts: '/*.jar, **/*.war', followSymlinks: false

                    slackSend channel: '#spring-petclinic',
                              message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} More info at: ${env.BUILD_URL}"
                }
            }
        }
        stage('Uploading artifacts'){
            steps{
                rtUpload (
                    serverId: 'Artifactory1',
                    spec: '''{
                          "files": [
                            {
                              "pattern": "**/*.war",
                              "target": "libs-release-local/"
                            }
                         ]
                    }''',
                    buildName: '${JOB_NAME}',
                    buildNumber: '${BUILD_NUMBER}'
                )
            }
            post{
                always{
                    rtPublishBuildInfo (
                        serverId: 'Artifactory-1',

                        buildName: '${JOB_NAME}',
                        buildNumber: '${BUILD_NUMBER}'
                    )
                }
            }
        }
        stage('Code-Analysis'){
            steps{
                withSonarQubeEnv('MySonarServer'){
                    mvn sonar:sonar
                }
            }
        }
    }
}
