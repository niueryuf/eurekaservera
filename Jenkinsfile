// 需要在jenkins的Credentials设置中配置jenkins-harbor-creds、jenkins-k8s-config参数
pipeline {
    agent any
    environment {
        GIT_TAG = sh(returnStdout: true,script: 'git describe --tags --always').trim()
		username = "admin"
		password = "Harbot12345"
    }
	 parameters {
        string(name: 'HARBOR_HOST', defaultValue: '47.106.224.69', description: 'harbor仓库地址')
        string(name: 'DOCKER_IMAGE', defaultValue: 'library/eurekaserver_jks', description: 'docker镜像名')
        string(name: 'APP_NAME', defaultValue: 'eurekaserver_jks', description: 'k8s中标签名')
        string(name: 'K8S_NAMESPACE', defaultValue: 'jinghang', description: 'k8s的namespace名称')
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
				 sh "docker login -u ${username} -p ${password}  ${params.HARBOR_HOST}"
                sh "docker build --build-arg JAR_FILE=`ls target/*.jar |cut -d '/' -f2` -t ${params.HARBOR_HOST}/${params.DOCKER_IMAGE}:${GIT_TAG} ."
                sh "docker push ${params.HARBOR_HOST}/${params.DOCKER_IMAGE}:${GIT_TAG}"
                sh "docker rmi ${params.HARBOR_HOST}/${params.DOCKER_IMAGE}:${GIT_TAG}"
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
