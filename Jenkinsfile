//pull config repo ok
// update tag in config repo ok
// not yet commit and push
def appSourceRepo = 'https://github.com/idiosyncrasy00/demo-app.git'
def appSourceBranch = 'master'

// def appConfigRepo = 'https://github.com/idiosyncrasy00/app-helmchart.git'
def appConfigBranch = 'master'
def helmRepo = "app-helmchart"
def demoApp = "demo-app"
def helmValueFile = "charts/app-demo-value.yaml"

def dockerhubAccount = 'dockerhub'
def githubAccount = 'github'

dockerBuildCommand = './'
def version = "v1.${BUILD_NUMBER}"

pipeline {
    agent any    
   
    environment {
        DOCKER_REGISTRY = 'https://registry-1.docker.io'
        DOCKER_IMAGE_NAME = "lockedaway/argo-cd-demo-app"
        DOCKER_IMAGE = "registry-1.docker.io/${DOCKER_IMAGE_NAME}"
    }

    stages {        
        stage('Checkout project') {
          steps {
            git branch: appSourceBranch,
                credentialsId: githubAccount,
                url: appSourceRepo
          }
        }
        stage('Build And Push Docker Image') {
            steps {
                script {
                    sh "git reset --hard"
                    sh "git clean -f"                    
					app = docker.build(DOCKER_IMAGE_NAME, dockerBuildCommand)
                    docker.withRegistry( DOCKER_REGISTRY, dockerhubAccount ) {
                       app.push(version)
                    }

                    sh "docker rmi ${DOCKER_IMAGE_NAME} -f"
                    sh "docker rmi ${DOCKER_IMAGE}:${version} -f"
                }
            }
        }

	stage('Update value in helm-chart') {
	    steps {
	        withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
	            sh """
	                git config --global user.email "jenkins@example.com"
	                git config --global user.name "jenkins"
	                sed -i 's|  tag: .*|  tag: "${version}"|' ${helmValueFile}
	                echo "Entering helm chart...."
	                git add ${helmValueFile}
	                git commit -m "Update to version ${version}"
	                git push https://x-access-token:${GITHUB_TOKEN}@github.com/idiosyncrasy00/demo-app.git
	                echo "After helm chart...."
	            """
	        }
	    }
	}
    }
}
