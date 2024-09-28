pipeline {
    agent any
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['default', 'dev', 'soap'], description: 'Selecciona el entorno de pruebas')
        string(name: 'TAGS', defaultValue: '', description: 'Dejar vacío para ejecutar todas las pruebas')
        choice(name: 'BROSWER', choices: ['chrome', 'edge', 'firefox'], description: 'Selecciona el navegador')
    }
    stages {
        stage('Prepare environment') {
            steps {
                sh 'env | sort'
                sh 'docker system df'
                sh "docker compose --project-name ${BUILD_TAG} up -d --quiet-pull"
                sh 'docker ps -a'
                sh 'docker network ls'
            }
        }
        stage('End2End Tests') {
            steps {
                script {
//                    sh 'sleep 2m'
//                    sh "docker run --rm -v ${WORKSPACE}:/usr/src/app -w /usr/src/app --name ${BUILD_TAG} --network ${BUILD_TAG}_default maven:3.8.8-eclipse-temurin-17 sleep 5m"
                    def cucumberFilterTags = TAGS?.trim() ? "-Dcucumber.filter.tags='${TAGS}'" : ""
                    sh "docker run --rm --user \$(id -u):\$(id -g) -v ${WORKSPACE}:/usr/src/app -w /usr/src/app --name ${BUILD_TAG} --network ${BUILD_TAG}_default maven:3.8.8-eclipse-temurin-17 mvn clean verify -Dwebdriver.remote.url=http://${BUILD_TAG}-selenium-hub-1:4444/wd/hub -Dwebdriver.remote.driver=${BROSWER} -Denvironment=${ENVIRONMENT} ${cucumberFilterTags} -B -ntp"

//                    sh 'chmod -R 755 target/site/serenity'

                    publishHTML(
                        target: [
                            reportName           : 'Serenity Report',
                            reportDir            : 'target/site/serenity',
                            reportFiles          : 'index.html',
                            keepAll              : true,
                            alwaysLinkToLastBuild: true,
                            allowMissing         : true
                        ]
                    )
                }
            }
        }
    }
    post {
        always {
            sh "docker compose --project-name ${BUILD_TAG} down --rmi all --volumes"
            sh 'docker builder prune -f'
            sh 'docker system df'
//            cleanWs()
        }
    }
}
