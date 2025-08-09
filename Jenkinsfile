pipeline {
    agent any

    environment {
        VENV_DIR = 'venv'
        GCP_PROJECT = 'mlops-udemy-467812'
        GCLOUD_PATH = '/var/jenkins_home/google-cloud-sdk/bin'
        LOG_DIR = 'logs'
    }

    stages {
        stage('Cloning Github repo to Jenkins') {
            steps {
                script {
                    echo 'Cloning Github repo to Jenkins...'
                    checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[
                        credentialsId: 'github_token', 
                        url: 'https://github.com/Jehoi-ga-ada/mlops-hotel-reservation.git'
                    ]])
                }
            }
        }

        stage('Setting up Virtual Environment and Installing dependencies') {
            steps {
                script {
                    echo 'Setting up Virtual Environment and Installing dependencies...'
                    sh '''
                    python -m venv ${VENV_DIR}
                    . ${VENV_DIR}/bin/activate
                    pip install --upgrade pip
                    pip install -e .
                    '''
                }
            }
        }

        stage('Building Docker Image') {
            steps {
                script {
                    echo 'Building Docker Image...'
                    sh '''
                    docker build -t gcr.io/${GCP_PROJECT}/ml-project:latest .
                    '''
                }
            }
        }

        stage('Pushing Docker Image to GCR') {
            steps {
                withCredentials([file(credentialsId : 'gcp-key', variable : 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    script {
                        echo 'Pushing Docker Image to GCR...'
                        sh '''
                        export PATH=$PATH:${GCLOUD_PATH}

                        gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
                        gcloud config set project ${GCP_PROJECT}
                        gcloud auth configure-docker --quiet

                        docker push gcr.io/${GCP_PROJECT}/ml-project:latest
                        '''
                    }
                }
            }
        }

        stage('Run Training in Container') {
            steps {
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    script {
                        echo 'Running training pipeline inside container...'
                        sh '''
                        docker run --rm \
                            -v ${GOOGLE_APPLICATION_CREDENTIALS}:/tmp/key.json \
                            -e GOOGLE_APPLICATION_CREDENTIALS=/tmp/key.json \
                            -v $(pwd)/${LOG_DIR}:/app/${LOG_DIR} \
                            gcr.io/${GCP_PROJECT}/ml-project:latest \
                            python pipeline/training_pipeline.py
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Archiving logs...'
            archiveArtifacts artifacts: "${LOG_DIR}/*.log", fingerprint: true
        }
    }
}
