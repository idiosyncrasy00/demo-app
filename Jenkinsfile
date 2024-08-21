//pull config repo ok
// update tag in config repo ok
// not yet commit and push
def appSourceRepo = 'https://github.com/idiosyncrasy00/demo-app.git'
def appSourceBranch = 'revert-branch'

def appConfigRepo = 'https://github.com/idiosyncrasy00/app-helmchart.git'
def appConfigBranch = 'master'
def helmRepo = "app-helmchart"
def helmChart = "app-demo"
def helmValueFile = "app-demo/app-demo-value.yaml"

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
        //GIT_COMMIT_HASH = '9ec07bc'  // Replace with the specific commit hash
    }

    stages {        
        stage('Checkout project') {
          steps {
            git branch: appSourceBranch,
                credentialsId: githubAccount,
                url: appSourceRepo
            // Checkout the specific commit
            //sh "git checkout ${GIT_COMMIT_HASH}"
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
			    sh """#!/bin/bash
					   [[ -d ${helmRepo} ]] && rm -r ${helmRepo}
					   git clone ${appConfigRepo} --branch ${appConfigBranch}
					   cd ${helmRepo}
					   sed -i 's|  tag: .*|  tag: "${version}"|' ${helmValueFile}
					   git add . ; git commit -m "Update to version ${version}";git push https://x-access-token:${GITHUB_TOKEN}@github.com/idiosyncrasy00/app-helmchart.git
					   cd ..
					   [[ -d ${helmRepo} ]] && rm -r ${helmRepo}
					   """	
				}				
            }
        }
    }
}
