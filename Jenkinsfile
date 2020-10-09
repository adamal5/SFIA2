pipeline{
        agent any
        options {
          skipDefaultCheckout true
        }
        environment {
            app_version = 'v1'
            rollback = 'false'
        }
        
          stages{      
            stage('Install Docker and Docker Compose'){
                steps{
                    sh '''
                    curl https://get.docker.com | sudo bash 
                    sudo usermod -aG docker $(whoami)
                    sudo apt update
                    sudo apt install -y curl jq
                    version=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | jq -r '.tag_name')
                    sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
                    sudo chmod +x /usr/local/bin/docker-compose
                    sudo chmod 666 /var/run/docker.sock
                    git clone https://github.com/adamal5/SFIA2 
                    cd SFIA2
                    '''
  
            }
        }
      
            stage('Build FrontImage'){
                steps{      
                    script{
                        dir("/SFIA2/frontend"){
                          if (env.rollback == 'false'){
                            frontendimage = docker.build("adamal5/sfia2-frontend")
                        }
                      }          
                    }
                }          
            }

            stage('Tag & Push FrontImage'){
                steps{
                    script{
                        if (env.rollback == 'false'){
                            docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials'){
                                frontendimage.push("${env.app_version}")    
                            }
                        }
                    }
                }          
            }                

            stage('Build BackImage'){
                steps{      
                    script{
                        dir("SFIA2/backend"){
                          if (env.rollback == 'false'){
                            backendimage = docker.build("adamal5/sfia2-backend")
                        }
                      }          
                    }
                }          
            }   
                
            stage('Tag & Push BackImages'){
                steps{
                    script{
                        if (env.rollback == 'false'){
                            docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials'){
                                backendimage.push("${env.app_version}")  
                            }
                        }
                    }
                }          
            } 
 
            stage('Build DatabaseImage'){
                steps{      
                    script{
                        dir("SFIA2/database"){
                          if (env.rollback == 'false'){
                            databaseimage = docker.build("adamal5/sfia2-database")
                        }
                      }          
                    }
                }          
            }                

            stage('Tag & Push DatabaseImage'){
                steps{
                    script{
                        if (env.rollback == 'false'){
                            docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials'){
                                databaseimage.push("${env.app_version}")    
                            }
                        }
                    }
                }          
            }

            stage('Install Docker and Docker Compose on App VM'){
                steps{
                    sh '''
                    ssh ubuntu@ip-172-31-24-23 -y <<EOF
                    curl https://get.docker.com | sudo bash 
                    sudo usermod -aG docker $(whoami)
                    sudo apt update
                    sudo apt install -y curl jq
                    version=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | jq -r '.tag_name')
                    sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
                    sudo chmod +x /usr/local/bin/docker-compose
                    sudo chmod 666 /var/run/docker.sock
EOF
                    '''
  
            }
        }            
        
            stage('Install mySQL client and Create table in Database'){
                steps{
                    sh '''
                    ssh ubuntu@ip-172-31-24-23 -y <<EOF
                    sudo apt update
                    sudo apt install mysql-client-core-5.7 -y
                    mysql -h terraform-20201009083922769400000001.cdsmwkad1q7o.eu-west-2.rds.amazonaws.com -P 3306 -u admin -p ab5gh78hj
                    CREATE DATABASE testdb;
                    CREATE DATABASE users;
                    USE users;

                    DROP TABLE IF EXISTS `users`;

                    CREATE TABLE `users` (
  `                   userName` varchar(30) NOT NULL
                    );

                    INSERT INTO `users` VALUES ('Bob'),('Jay'),('Matt'),('Ferg'),('Mo');
                    exit 
EOF
                    '''
  
            }
        }      
            
                  
            stage('Deploy App'){
                steps{    
                    sh '''
                    ssh ubuntu@ip-172-31-24-23 -y <<EOF
                    git clone https://github.com/adamal5/SFIA2
                    cd SFIA2
                    docker pull adamal5/sfia2-frontend:v1
                    docker pull adamal5/sfia2-backend:v1
                    docker pull adamal5/sfia2-database:v1
                    
                    docker-compose up -d
EOF
                    '''
                    }
                }   
            stage('Run Frontend Test'){
                steps{
                    sh '''
                    ssh ubuntu@ip-172-31-24-23 -y <<EOF
                    sleep 15
                    cd SFIA2/frontend/tests
                    docker-compose exec -T frontend pytest --cov application > frontend-test.txt
EOF
                    '''
                    }
                }                 
            stage('Run Backend Test'){
                steps{
                    sh '''
                    ssh ubuntu@ip-172-31-24-23 -y <<EOF
                    cd SFIA2/frontend/tests
                    docker-compose exec -T backend pytest --cov application > backend-test.txt

EOF
                    '''
                    }
                }                 
            }
}
