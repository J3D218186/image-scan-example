pipeline {
    
    agent any
    
    environment {
        FALCON_CLIENT_SECRET = credentials('FALCON_CLIENT_SECRET')
        FALCON_CLIENT_ID = credentials('FALCON_CLIENT_ID')
        BUILD_DIR = '.'
        CONTAINER_REPO = 'yasoniayp/image-scan-example'
        CONTAINER_TAG = "${BUILD_NUMBER}"
        FALCON_CLOUD_REGION = 'us-2'   
    }
    
    stages {
        stage('BuildImage') {
            // Build the docker image using the docker file in the current directory
            steps{
                sh "docker build -t $CONTAINER_REPO:$CONTAINER_TAG $BUILD_DIR"
            }
        }
        // stage('ScanImage') {
        //     steps {
        //         echo "Scanning image: ${CONTAINER_REPO}:${CONTAINER_TAG}"
        //         script {
        //             def scanResponse = crowdStrikeSecurity(
        //                 imageName: "${CONTAINER_REPO}",
        //                 imageTag: "${CONTAINER_TAG}",
        //                 enforce: true,
        //                 timeout: 60
        //             )
        //             echo "Scan Result: ${scanResponse}"
                    
        //             def criticalVulns = scanResponse.issues?.findAll { it.severity == 'Critical' }
        //             if (criticalVulns?.size() > 0) {
        //                 error "Build failed due to Critical vulnerabilities found!"
        //             }
        //         }
        //     }
        // }

        stage('ScanImage') {
          steps {
            echo "Scanning image: ${CONTAINER_REPO}:${CONTAINER_TAG}"
            crowdStrikeSecurity imageName: "${CONTAINER_REPO}",
                                imageTag: "${CONTAINER_TAG}",
                                enforce: true,
                                timeout: 60
          }
        }
        stage('PushImage') {
            // Push the container image
            steps{
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId:'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                sh '''
                echo $PASSWORD | docker login -u $USERNAME --password-stdin
                docker push $CONTAINER_REPO:$CONTAINER_TAG
                '''
                }
            }
        }  
    }
}
