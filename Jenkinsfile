def gv

pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    stages {
        stage('increment version') {
            steps {
                script {
                    echo 'incrementing app version...'
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }
        stage('build app') {
            steps {
                script {
                    echo 'building the application...'
                    sh 'mvn clean package'
                }
            }
        }
        stage('build image') {
            steps {
                script {
                    echo "building the docker image..."
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]){
                        sh "docker build -t {PRIVATE_REPO}:${IMAGE_NAME} ."
                        sh 'echo $PASS | docker login -u $USER --password-stdin'
                        sh "docker push {PRIVATE_REPO}:${IMAGE_NAME}"
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                script {
                    echo 'deploying docker image...'
                }
            }
        }
        stage('commit version update') {
    steps {
        script {
            withCredentials([usernamePassword(
                credentialsId: 'github-credentials',
                passwordVariable: 'PASS',
                usernameVariable: 'USER'
            )]) {

                sh 'git config user.email "jenkins@example.com"'
                sh 'git config user.name "jenkins"'

                sh 'git add pom.xml'

                sh '''
                    git diff --cached --quiet || \
                    git commit -m "ci: version bump"
                '''

                sh '''
                    git -c credential.helper='!f() { echo "$PASS"; }; f' \
                    push https://$USER@github.com/GeraldAdugbo/java-maven-app.git \
                    HEAD:jenkins-jobs
                '''
            }
        }
    }
}
