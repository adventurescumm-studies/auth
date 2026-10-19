pipeline {
    // Agente Linux com Git, JDK 25 e Maven no PATH.
    agent any

    options {
        disableConcurrentBuilds()
        timeout(time: 30, unit: 'MINUTES')
    }

    environment {
        // As dependencias e o projeto usam o mesmo repositorio Maven por job.
        MAVEN_REPO = "${env.WORKSPACE}/.m2/repository"
    }

    stages {
        stage('Dependencies - account') {
            steps {
                dir('.jenkins-deps/account') {
                    deleteDir()
                    git branch: 'main', url: 'https://github.com/adventurescumm-studies/account.git'
                    sh 'mvn -B -ntp -Dmaven.repo.local="$MAVEN_REPO" clean install'
                }
            }
        }

        stage('Build & Install') {
            steps {
                // auth e uma biblioteca: install disponibiliza o JAR e o POM localmente.
                sh 'mvn -B -ntp -Dmaven.repo.local="$MAVEN_REPO" clean install'
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'target/*.jar,pom.xml', fingerprint: true
        }
    }
}
