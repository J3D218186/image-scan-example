pipeline {
    
    agent any
    
    environment {
        // Replace the <tags> below with your values.
        FALCON_CLIENT_SECRET = credentials('FALCON_CLIENT_SECRET')
        FALCON_CLIENT_ID = credentials('FALCON_CLIENT_ID')
        BUILD_DIR = '.'
        CONTAINER_REPO = 'maleakhiek/image-scan-example'
        CONTAINER_TAG = "${BUILD_NUMBER}"
        FALCON_CLOUD_REGION = 'us-2'
        //SCORE = 500
    }
    
    stages {
        stage('BuildImage') {
            // Build the docker image using the docker file in the current directory
            steps{
                sh "docker build -t $CONTAINER_REPO:$CONTAINER_TAG $BUILD_DIR"
            }
        }
        stage('ScanImage') {
            // Scan the image using Falcon ImageScan API
            steps {
                sh '''
                if [ ! -d container-image-scan ]; then
                    git clone https://github.com/crowdstrike/container-image-scan
                fi
        
                # Update package list and install necessary packages
                apt update && apt install -y python3-pip python3-venv pipx
        
                # Ensure pipx is set up correctly
                export PATH=$PATH:/root/.local/bin
        
                # Create and activate a virtual environment
                python3 -m venv venv
                . venv/bin/activate
        
                # Install required Python packages
                pip install docker crowdstrike-falconpy retry
        
                # Run the scan script using the virtual environment
                python3 container-image-scan/cs_scanimage.py -s 999999
                '''
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
