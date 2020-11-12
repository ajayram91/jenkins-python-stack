pipeline{
    agent any
    environment{
        MYSQL_DATABASE_HOST = "database-42.cbanmzptkrzf.us-east-1.rds.amazonaws.com"
        MYSQL_DATABASE_PASSWORD = "Clarusway"
        MYSQL_DATABASE_USER = "admin"
        MYSQL_DATABASE_DB = "phonebook"
        MYSQL_DATABASE_PORT = 3306
        PATH="/usr/local/bin/:${env.PATH}"
    }
    stages{
       stage("compile"){
           agent{
               docker{
                   image 'python:alpine'
               }
           }
           steps{
               withEnv(["HOME=${env.WORKSPACE}"]) {
                    sh 'pip install -r requirements.txt'
                    sh 'python -m py_compile src/*.py'
                    stash(name: 'compilation_result', includes: 'src/*.py*')
                }
           }
       }
       stage('test') {
            agent {
                docker {
                    image 'python:alpine'
                }
            }
            steps {
                withEnv(["HOME=${env.WORKSPACE}"]) {
                    sh 'python -m pytest -v --junit-xml results.xml src/appTest.py'
                }
            }
            post {
                always {
                    junit 'results.xml'
                }
            }
        }
        stage('build'){
            agent any
            steps{
                sh "docker build -t ajay-phonebook-app/to-do-repo ."
                sh "docker tag ajay-phonebook-app/to-do-repo:latest 188358726447.dkr.ecr.us-east-1.amazonaws.com/ajay-phonebook-app/to-do-repo:latest"
            }
        }
//Inserting comment for new Build Push
        stage('push'){
            agent any
            steps{
                sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 188358726447.dkr.ecr.us-east-1.amazonaws.com/ajay-phonebook-app/to-do-repo"
                sh "docker push 188358726447.dkr.ecr.us-east-1.amazonaws.com/ajay-phonebook-app/to-do-repo:latest"
            }
        }

        stage('compose'){
            agent any
            steps{
               sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 188358726447.dkr.ecr.us-east-1.amazonaws.com"
               sh "docker-compose up -d"
            }
        }
        stage('get-keypair'){
            agent any
            steps{
                sh '''
                    if [ -f "ajayKey10_public.pem" ]
                    then
                        echo "file exists..."
                    else
                        aws ec2 create-key-pair \
                          --region us-east-1 \
                          --key-name ajayKey7.pem \
                          --query KeyMaterial \
                          --output text > ajayKey7.pem
                        chmod 400 ajayKey7.pem
                        ssh-keygen -y -f ajayKey7.pem >> ajayKey10_public.pem
                    fi
                '''
              }
          }  
         
          stage('create-cluster'){
            agent any
            steps{
                sh '''
                    #!/bin/sh
                    running=$(sudo lsof -i:80) || true
                    if [ "$running" != '' ]
                    then
                        docker-compose down
                        exist="$(aws eks list-clusters | grep ajays-cluster2)" || true
                        if [ "$exist" == '' ]
                        then
                            eksctl create cluster \
                                --name ajays-cluster2 \
                                --version 1.18 \
                                --region us-east-1 \
                                --nodegroup-name my-nodes \
                                --node-type t2.small \
                                --nodes 1 \
                                --nodes-min 1 \
                                --nodes-max 2 \
                                --ssh-access \
                                --ssh-public-key  ajayKey10_public.pem \
                                --managed
                        else
                            echo 'no need to create cluster...'
                        fi
                    else
                        echo 'app is not running with docker-compose up -d'
                    fi
                '''
            }
         } 



        }

    }
