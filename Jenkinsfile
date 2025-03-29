@Library('shared')_

pipeline{
    agent {label 'dev-server'}
    
    stages{
        stage("Code"){
            steps{
                script{
                clone("https://github.com/Ritik200w/Springboot-BankApp.git","main")
                echo "Code clonning done."
                }
            }
        }
        stage("Build"){                                                             
            steps{
                script{
                dockerbuild("bankapp-mini","latest")
                echo "Code build bhi hogaya."
                }
            }
        }
        stage("Push to DockerHub"){
            steps{
                script{
                dockerpush("dockerHub","bankapp-mini","latest")
                echo "Push to dockerHub is also done."
                }
            }
        }
        stage("Deplying"){
            steps{
                script{
                deploy()
                echo "Deployment bhi done."
                }
            }
        }
    }
}
