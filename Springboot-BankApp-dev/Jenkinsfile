@Library('Shared')_

pipeline{
    agent {label 'dev-server'}
    
    stages{
        stage("Code"){
            steps{
                clone("https://github.com/Amitabh-DevOps/banking-app-project.git","dev")
                echo "Code clonning done."
            }
        }
        stage("Build"){                                                             
            steps{
                dockerbuild("bankapp-mini","latest")
                echo "Code build bhi hogaya."
            }
        }
        stage("Push to DockerHub"){
            steps{
                dockerpush("dockerHub","bankapp-mini","latest")
                echo "Push to dockerHub is also done."
            }
        }
        stage("Deplying"){
            steps{
                deploy()
                echo "Deployment bhi done."
            }
        }
    }
}
