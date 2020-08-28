pipeline{
    agent any
    environment {
        GIT_TAG = sh(returnStdout: true,script: 'git describe --tags --always').trim()
		username = "niueryuf"
		password = "csy98314.."
    }
    parameters {
        string(name: 'HARBOR_HOST', defaultValue: '172.23.101.66', description: 'harbor仓库地址')
        string(name: 'DOCKER_IMAGE', defaultValue: 'tssp/pipeline-demo', description: 'docker镜像名')
        string(name: 'APP_NAME', defaultValue: 'pipeline-demo', description: 'k8s中标签名')
        string(name: 'K8S_NAMESPACE', defaultValue: 'demo', description: 'k8s的namespace名称')
    }
    stages{
        stage("Maven Build"){
            steps{
                sh 'mvn clean package -Dfile.encoding=UTF-8 -DskipTests=true'
                stash includes: 'target/*.jar', name: 'app'
            }
        }
		stage('Docker Build'){
			steps {
                unstash 'app'
                sh "docker login -u ${username} -p ${password}"
                sh "docker build --build-arg JAR_FILE=`ls target/*.jar |cut -d '/' -f2` -t ${params.HARBOR_HOST}/${params.DOCKER_IMAGE}:${GIT_TAG} ."
                sh "docker push ${username}/${params.DOCKER_IMAGE}:${GIT_TAG}"
            }
		}
		
    }
}
