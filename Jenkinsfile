pipeline{
    agent any

    environment{
        VENV_DIR = 'venv'
    }

    stages{
        stage('Cloning Github repo to Jenkins'){
            steps{
                script{
                    echo 'Cloning Github repo to Jenkins...'
                    checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github_token', url: 'https://github.com/Jehoi-ga-ada/mlops-hotel-reservation.git']])
                }
            }
        }

        stage('Setting up Virtual Environment and Installing dependencies'){
            steps{
                script{
                    echo 'Setting up Virtual Environment and Installing dependencies...'
                    sh '''
                    python -m ${VENV_DIR}
                    . ${VENV_DIR}/bin/activate
                    pip install --upgrade pip
                    pip install -e .
                    '''
                }
            }
        }
    }
}