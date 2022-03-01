pipeline {
    agent {
        label 'OneS'
    }
    environment {
        envString = 'true'
    }
    post {
        always {
            allure includeProperties: false, jdk: '', results: [[path: 'build/syntax-check/allure'], [path: 'build/smoke/allure'], [path: 'build/tests/allure'], [path: 'build/vanessa/allure']]
            junit allowEmptyResults: true, testResults: 'build/syntax-check/junit/junit.xml'
            junit allowEmptyResults: true, testResults: 'build/smoke/junit/*.xml'
            junit allowEmptyResults: true, testResults: 'build/tests/junit/*.xml'
            junit allowEmptyResults: true, testResults: 'build/vanessa/junit/*.xml'
        }
        failure {
            bat "echo failure"
        }
        success {
            bat "echo success"
        }
    }
    stages {
        stage("Создание тестовой базы") {
            steps {
                bat "chcp 65001\n vrunner init-dev --dt C:\\devops\\course.dt --db-user Администратор --src src/cf"
            }
        }
        stage("Синтаксический контроль") {
            steps {
                script {
                    try {
                        bat "chcp 65001\n vrunner syntax-check"
                    } catch(Exception Exc) {
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }
        stage("Дымовые тесты") {
            steps {
                script {
                    try {
                        bat "chcp 65001\n vrunner xunit"
                    } catch(Exception Exc) {
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }
        stage("Модульные тесты") {
            steps {
                script {
                    try {
                        bat """chcp 65001
                        call vrunner compileepf tests tests
                        call vrunner xunit --settings ./env-tests.json"""
                    } catch(Exception Exc) {
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }
        stage("vanessa") {
            steps {
                script {
                    try {
                        bat "chcp 65001\n vrunner vanessa"
                    } catch(Exception Exc) {
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }
        stage("АПК") {
            steps {
                script {
                    try {
                        bat "chcp 65001\n vrunner run --ibconnection /FC:/devops/acc --db-user \"\" --db-pwd \"\" --command \"acc.catalog=${WORKSPACE};acc.propertiesPaths=./tools/acc-export/acc.properties;\" --execute \"./tools/acc-export/acc-export.epf\" --ordinaryapp 1"
                    } catch(Exception Exc) {
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }
        stage("Sonar") {
            steps {
                script {
                    scannerHome = tool name: 'sonar-scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                }
                withSonarQubeEnv("Sonar") {
                    bat "${scannerHome}/bin/sonar-scanner -Dsonar.projectVersion=${BUILD_ID}"
                }
            }
        }
    }
}