#!groovy
//  groovy Jenkinsfile
properties([disableConcurrentBuilds()])\

pipeline  {
        agent { 
           label ''
        }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
        timestamps()
    }
    stages {
       
        stage("base") {
            steps {
                sh '''
                rm -rf /var/lib/zabbix/
                mkdir /var/lib/zabbix/
                cd /var/lib/zabbix/
                ln -s /usr/share/zoneinfo/Europe/Kiev localtime
                echo 'Europe/Kiev' > timezone
                '''
            } 
        }
        stage("network") {
            steps {
                sh '''
                echo "test"
                docker network rm zabbix-net
                docker stop $(docker ps -q)
                docker systen prune -a
                docker network create zabbix-net            
                '''
            }
        }
          stage("Postgresql") {
            steps {
                sh '''
                docker run -d \
                --name zabbix-postgres \
                --network zabbix-net \
                -v /var/lib/zabbix/timezone:/etc/timezone \
                -v /var/lib/zabbix/localtime:/etc/localtime \
                -e POSTGRES_PASSWORD=zabbix \
                -e POSTGRES_USER=zabbix \
                -d postgres:alpine
                   docker pull yurashupik/zabbix:1
                '''
            }
          }
        stage("Zabbix server") {
            steps {
                sh '''
                docker run \
                --name zabbix-server \
                --network zabbix-net \
                -v /var/lib/zabbix/alertscripts:/usr/lib/zabbix/alertscripts \
                -v /var/lib/zabbix/timezone:/etc/timezone \
                -v /var/lib/zabbix/localtime:/etc/localtime \
                -p 10051:10051 -e DB_SERVER_HOST="zabbix-postgres" \
                -e POSTGRES_USER="zabbix" \
                -e POSTGRES_PASSWORD="zabbix" \
                -d zabbix/zabbix-server-pgsql:alpine-latest
                '''
            }
        }
        stage("Zabbix web server") {
            steps {
                sh '''
                docker run \
                --name zabbix-web \
                -p 80:8080 -p 443:8443 \
                --network zabbix-net \
                -e DB_SERVER_HOST="zabbix-postgres" \
                -v /var/lib/zabbix/timezone:/etc/timezone \
                -v /var/lib/zabbix/localtime:/etc/localtime \
                -e POSTGRES_USER="zabbix" \
                -e POSTGRES_PASSWORD="zabbix" \
                -e ZBX_SERVER_HOST="zabbix-server" \
                -e PHP_TZ="Europe/Kiev" \
                -d zabbix/zabbix-web-nginx-pgsql:alpine-latest
                '''
            }
        }
        stage("run") {
            steps {
                sh '''
                docker start zabbix-postgres
                docker start zabbix-server
                docker start zabbix-web
                '''
            }
        }
    
                
        stage("docker login") {
            steps {
                echo " ============== docker login =================="
                withCredentials([usernamePassword(credentialsId: '1ca9b926-1985-4c5d-9e8b-dc8cb2ed45b6', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        def loginResult = sh(script: "docker login -u $USERNAME -p $PASSWORD", returnStatus: true)
                        if (loginResult != 0) {
                            error "Failed to log in to Docker Hub. Exit code: ${loginResult}"
                        }
                    }
                }
                echo " ============== docker login completed =================="
            }
        }
       
        stage("docker push") {
            steps {
                echo " ============== pushing image =================="
                sh '''
                docker tag zabbix/zabbix-web-nginx-pgsql:alpine-latest dimax555/mnm221:zabbix-web1
                docker tag zabbix/zabbix-server-pgsql:alpine-latest dimax555/mnm221:zabbix-server1
                docker tag postgres:alpine dimax555/mnm221:zabbix-postgres1
                docker push dimax555/mnm221:zabbix-postgres1
                docker push dimax555/mnm221:zabbix-server1
                docker push dimax555/mnm221:zabbix-web1
                '''
            }
        }
               stage("remove") {
            steps {
                echo " ============== remune =================="
                sh '''
                docker stop $(docker ps -q) && docker rm $(docker ps -a -q)
                '''
            }
        }    
            stage("docker pull") {
            steps {
                echo " ============== pushing image =================="
                sh '''
               docker pull dimax555/mnm221:zabbix-postgres1
               docker pull dimax555/mnm221:zabbix-server1
               docker pull dimax555/mnm221:zabbix-web1
                '''
            }
        }
          stage("docker run") {
            steps {
                echo " ============== pushing image =================="
                sh '''
               docker run dimax555/mnm221:zabbix-postgres1
               docker run  dimax555/mnm221:zabbix-server1
               docker run dimax555/mnm221:zabbix-web1
                '''
            }
        }  
    }
}
