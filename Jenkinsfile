pipeline{
    agent any
    stages{
        stage("Maven Build"){
            steps{
                sh 'mvn clean package -Dfile.encoding=UTF-8 -DskipTests=true'
                stash includes: 'target/*.jar', name: 'app'
            }
        }
    }
}
