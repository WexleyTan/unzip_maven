pipeline {
    agent any
    environment {
        IMAGE = "neathtan/gradle17_clone"
        FILE_NAME = "latest_maven.zip"
        DIR_UNZIP = "latest_maven"  
        DOCKER_IMAGE = "${IMAGE}:${BUILD_NUMBER}"
        DOCKER_CONTAINER = "springboot17_jenkins"
        DOCKER_CREDENTIALS_ID = "dockertoken"
        GIT_BRANCH = "master"
        GIT_MANIFEST_REPO = "https://github.com/WexleyTan/gradle17_manifest.git"
        MANIFEST_REPO = "gradle17_manifest"
        MANIFEST_FILE_PATH = "deployment.yaml"
        GIT_CREDENTIALS_ID = 'git_pass'
    }

    stages {

        stage('Unzip File') {
            steps {
                script {
                    echo "Checking if the file ${FILE_NAME} exists and unzipping it if present..."
                    sh """
                        if [ -f '${FILE_NAME}' ]; then
                            echo "Removing existing directory ${DIR_UNZIP}..."
                            rm -rf ${DIR_UNZIP}
                            echo "Unzipping the file..."
                            unzip -o '${FILE_NAME}'
                        else
                            echo "'${FILE_NAME}' does not exist."
                            exit 1
                        fi
                    """
                    sh "ls"
                }
            }
        }

        stage('Create Dockerfile') {
            steps {
                script {
                    echo "Creating Dockerfile..."
                    writeFile file: 'Dockerfile', text: '''
                    FROM maven:3.8.7-eclipse-temurin-19 AS build
                    WORKDIR /app
                    COPY . .
                    RUN mvn clean package
                    FROM eclipse-temurin:22.0.1_8-jre-ubi9-minimal
                    COPY --from=build /app/target/*.jar /app/app.jar
                    EXPOSE 9090
                    ENTRYPOINT ["java", "-jar", "app.jar"]
                    '''
                }
            }
        }

        stage("Clean Package") {
            steps {
                script {
                    echo "Building the application..."
                    dir("${DIR_UNZIP}") {  
                        sh 'mvn clean install' 
                    }
                }
            }
        }
        
        stage("Build and Push Docker Image") {
            steps {
                script {
                    echo "Building Docker image..."
                    dir("${DIR_UNZIP}") { 
                        sh "docker build -t ${DOCKER_IMAGE} . > build_output.log 2>&1 "  
                    }
                    sh "docker images | grep -i ${IMAGE}"

                    echo "Pushing the image to Docker Hub..."
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }

        stage("Cloning the Manifest File") {
            steps {
                script {
                    echo "Checking if the manifest repository exists and removing it if necessary..."
                    sh """
                        if [ -d "${MANIFEST_REPO}" ]; then
                            echo "Directory ${MANIFEST_REPO} exists, removing it..."
                            rm -rf ${MANIFEST_REPO}
                        fi
                    """
                    
                    echo "Cloning the manifest repository..."
                    sh "git clone -b ${GIT_BRANCH} ${GIT_MANIFEST_REPO} ${MANIFEST_REPO}"
                }
            }
        }
        
        stage("Updating the Manifest File") {
            steps {
                script {
                    echo "Updating the image in the deployment manifest..."
                    dir("${MANIFEST_REPO}") {
                        sh """
                            sed -i 's|image: ${IMAGE}:.*|image: ${DOCKER_IMAGE}|' ${MANIFEST_FILE_PATH}
                            echo "Updated deployment file:"
                        """
                        
                        echo "Committing and pushing changes to the manifest repository..."
                        withCredentials([usernamePassword(credentialsId: GIT_CREDENTIALS_ID, passwordVariable: 'GIT_PASS', usernameVariable: 'GIT_USER')]) {
                            sh """
                                git config --global user.name "WexleyTan"
                                git config --global user.email "neathtan1402@gmail.com"
                                git add ${MANIFEST_FILE_PATH}
                                git commit -m "Update image to ${DOCKER_IMAGE}"
                                git push https://${GIT_USER}:${GIT_PASS}@github.com/WexleyTan/gradle17_manifest.git ${GIT_BRANCH}
                            """
                        }
                    }
                }
            }
        }
    }
}
