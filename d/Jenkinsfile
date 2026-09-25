pipeline {
    agent any

    options {
        timestamps()
    }

    stages {
        stage('Install dependencies') {
            steps {
                sh '''
                    python3 -m venv .venv
                    . .venv/bin/activate
                    pip install --upgrade pip
                    pip install ansible molecule molecule-plugins[docker] ansible-lint yamllint
                    ansible-galaxy install -r requirements.yml
                '''
            }
        }

        stage('Lint') {
            steps {
                sh '''
                    . .venv/bin/activate
                    yamllint .
                    ansible-lint
                '''
            }
        }

        stage('Test roles (Molecule)') {
            steps {
                sh '''
                    . .venv/bin/activate
                    cd roles/docker
                    molecule test
                '''
            }
        }

        stage('Deploy to staging') {
            when {
                branch 'main'
            }
            steps {
                sh '''
                    . .venv/bin/activate
                    ansible-playbook -i inventories/staging/hosts.yml site.yml
                '''
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
