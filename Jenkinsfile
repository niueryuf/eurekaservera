pipeline{
    agent any
	environment {
        GIT_TAG = sh(returnStdout: true,script: 'git describe --tags --always').trim()
		username = "niueryuf"
		password = "csy98314.."
    }
	 parameters {
        string(name: 'HARBOR_HOST', defaultValue: '172.23.101.66', description: 'harbor仓库地址')
        string(name: 'DOCKER_IMAGE', defaultValue: 'pipeline-demo', description: 'docker镜像名')
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
                sh "docker build --build-arg JAR_FILE=`ls target/*.jar |cut -d '/' -f2` -t ${username}/${params.DOCKER_IMAGE}:${GIT_TAG} ."
                sh "docker push ${username}/${params.DOCKER_IMAGE}:${GIT_TAG}"
                sh "docker rmi ${username}/${params.DOCKER_IMAGE}:${GIT_TAG}"
            }
		}
		stage('Deploy') {
		 steps {
                sh "sed -e 's#{IMAGE_URL}#${username}/${params.DOCKER_IMAGE}#g;s#{IMAGE_TAG}#${GIT_TAG}#g;s#{APP_NAME}#${params.APP_NAME}#g;s#{SPRING_PROFILE}#k8s-test#g' k8s-deployment.tpl > k8s-deployment.yml"
                sh "kubectl apply -f k8s-deployment.yml --namespace=${params.K8S_NAMESPACE}"
            }
		}
		
    }
}
