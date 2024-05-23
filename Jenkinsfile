pipeline {
    agent any

    parameters {
        choice(name: 'TerraformAction', choices: 'Deploy\nDestroy', description: 'Select the action to perform')
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/master']],
                        userRemoteConfigs: [[url: 'https://github.com/devopscoacht/capstone.git']]
                    ])
                }
            }
        }

        stage('Terraform Init and Plan') {
            when {
                expression {
                    return params.TerraformAction == 'Deploy'
                }
            }
            steps {
                script {
                    sh 'terraform init'
                    sh 'terraform plan -out=tfplan'
                }
            }
        }

        stage('Terraform Apply') {
            when {
                expression {
                    return params.TerraformAction == 'Deploy'
                }
            }
            steps {
                withAWS(credentials: 'aws-credentials', region: 'us-east-1') {
                    sh 'terraform apply -auto-approve tfplan'
                }
            }
        }

        stage('Terraform Destroy') {
            when {
                expression {
                    return params.TerraformAction == 'Destroy'
                }
            }
            steps {
                withAWS(credentials: 'aws-credentials', region: 'us-east-1') {
                    sh 'terraform destroy -auto-approve'
                }
            }
        }
    }

    post {
        success {
            script {
                if (params.TerraformAction == 'Deploy') {
                    echo 'Deployment succeeded'
                } else {
                    echo 'Destroy succeeded'
                }
            }
        }
        failure {
            script {
                if (params.TerraformAction == 'Deploy') {
                    echo 'Deployment failed'
                } else {
                    echo 'Destroy failed'
                }
            }
        }
    }
}
