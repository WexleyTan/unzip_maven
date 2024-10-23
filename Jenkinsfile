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
        
        // stage("Clean Package") {
        //     steps {
        //         script {
        //             echo "Building the application..."
        //             dir("${DIR_UNZIP}") {  
        //                 sh 'mvn clean install' 
        //             }
        //         }
        //     }
        // }
        
        stage("Build and Push Docker Image") {
            steps {
                script {
                    echo "Building Docker image..."
                    dir("${DIR_UNZIP}") { 
                        sh "docker build -t ${DOCKER_IMAGE} . > build_output.log 2>&1 "  
                    }
                    sh "docker images | grep -i ${IMAGE}"

    
                    echo "Logging in to Docker Hub using Jenkins credentials..."
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    }
                    
                    echo "Pushing the image to Docker Hub..."
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }
    }
}
