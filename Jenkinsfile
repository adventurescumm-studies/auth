// Premissa: auth e uma biblioteca Maven. Confirme no pom.xml.
// Nao publica imagem Docker nem artefatos em um repositorio Maven remoto.
pipeline {
    agent any // O agente selecionado precisa de Linux, Git e um JDK compativel.

    options {
        skipDefaultCheckout(true)
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '20'))
        timeout(time: 30, unit: 'MINUTES')
    }

    parameters {
        booleanParam(name: 'SKIP_TESTS', defaultValue: true,
                     description: 'Mantem o comportamento dos exemplos; desmarque para executar os testes Maven.')
    }

    stages {
        stage('Checkout') {
            steps {
                deleteDir()
                checkout scm
                script {
                    // Exporta explicitamente o parametro e preserva false.
                    env.SKIP_TESTS = (params.SKIP_TESTS == null ? true : params.SKIP_TESTS).toString()
                    env.MAVEN_REPO = "${pwd(tmp: true)}/maven-repository"
                }
                sh '''
                    set -eu
                    test -f pom.xml || { echo 'Esperado pom.xml na raiz do repositorio auth.' >&2; exit 1; }
                    java -version
                '''
            }
        }

        stage('Build e Install Maven') {
            steps {
                sh '''
                    set -eu
                    if [ -f ./mvnw ]; then
                        sh ./mvnw -B "-Dmaven.repo.local=$MAVEN_REPO" "-DskipTests=${SKIP_TESTS:-true}" clean install
                    else
                        mvn -B "-Dmaven.repo.local=$MAVEN_REPO" "-DskipTests=${SKIP_TESTS:-true}" clean install
                    fi
                '''
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: '**/target/*.jar,**/target/*.war',
                             fingerprint: true, allowEmptyArchive: true
        }
    }
}
