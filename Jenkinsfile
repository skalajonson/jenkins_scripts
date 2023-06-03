pipeline {
    agent any

    stages {
        stage('Creating directory for Docker images') {
            steps {
                sh '''
                mkdir -p /home/kaka/zabbix/ /home/kaka/zabbix/postgresql/ /home/kaka/zabbix/server/ /home/kaka/zabbix/web/
                '''
            }
        }
        stage('Creating Dockerfiles') {
            steps {
                sh '''
                touch /home/kaka/zabbix/postgresql/Dockerfile
                touch /home/kaka/zabbix/server/Dockerfile
                touch /home/kaka/zabbix/web/Dockerfile
                '''
            }
        }
        stage('Configure Dockerfiles') {
            steps {
                sh '''
                echo FROM chikibevchik/zabbix:postgres > /home/kaka/zabbix/postgresql/Dockerfile
                echo FROM chikibevchik/zabbix:server > /home/kaka/zabbix/server/Dockerfile
                echo FROM chikibevchik/zabbix:web > /home/kaka/zabbix/web/Dockerfile
                '''
            }
        }
        stage('Build docker images') {
            steps {
                sh '''
                cd /home/kaka/zabbix/postgresql/
                docker build -t chikibevchik/zabbixbyjenkins:postgres .
                cd /home/kaka/zabbix/server/
                docker build -t chikibevchik/zabbixbyjenkins:server .
                cd /home/kaka/zabbix/web/
                docker build -t chikibevchik/zabbixbyjenkins:web .
                '''
            }
        }
        stage("docker login") {
            steps {
                echo " ============== docker login =================="
                withCredentials([usernamePassword(credentialsId: 'DockerHub-Credentials', usernameVariable: 'chikibevchik', passwordVariable: 'topesto777')]) {
                    sh '''
                    docker login -u chikibevchik -p topesto777
                    '''
                }
            }
        }
            stage('docker push') {
                steps {
                    sh '''
                    docker push chikibevchik/zabbixbyjenkins:postgres
                    docker push chikibevchik/zabbixbyjenkins:server
                    docker push chikibevchik/zabbixbyjenkins:web
                    '''
                }
            }
            stage('Delete images') {
                steps {
                    sh '''
                    docker rmi --force chikibevchik/zabbixbyjenkins:postgres
                    docker rmi --force chikibevchik/zabbixbyjenkins:server
                    docker rmi --force chikibevchik/zabbixbyjenkins:web
                    '''
                }
            }
            stage('pull docker images') {
                steps {
                    sh '''
                    docker pull chikibevchik/zabbixbyjenkins:postgres
                    docker pull chikibevchik/zabbixbyjenkins:server
                    docker pull chikibevchik/zabbixbyjenkins:web
                    '''
                }
            }
            stage('Configure region and docker network') {
                steps {
                    sh '''
                    mkdir /var/lib/zabbix/ && cd /var/lib/zabbix/ && ln -s /usr/share/zoneinfo/Europe/Kiev localtime && echo 'Europe/Kiev' > timezone && sudo docker network create zabbix-net
                    '''
                }
            }
            stage('Install and run zabbix postgresql') {
                steps {
                    sh '''
                    sudo docker run --restart=always -d \
--name zabbix-postgres \
--network zabbix-net \
-v /var/lib/zabbix/timezone:/etc/timezone \
-v /var/lib/zabbix/localtime:/etc/localtime \
-e POSTGRES_PASSWORD=zabbix \
-e POSTGRES_USER=zabbix \
-d chikibevchik/zabbixbyjenkins:postgres
                    '''
                }
            }
            stage('install and run zabbix server') {
                steps {
                    sh '''
                    sudo docker run --restart=always \
--name zabbix-server \
--network zabbix-net \
-v /var/lib/zabbix/alertscripts:/usr/lib/zabbix/alertscripts \
-v /var/lib/zabbix/timezone:/etc/timezone \
-v /var/lib/zabbix/localtime:/etc/localtime \
-p 10051:10051 -e DB_SERVER_HOST="zabbix-postgres" \
-e POSTGRES_USER="zabbix" \
-e POSTGRES_PASSWORD="zabbix" \
-d chikibevchik/zabbixbyjenkins:server
                    '''
                }
            }
            stage('install and run zabbix web') {
                steps {
                    sh '''
                    sudo docker run --restart=always \
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
-d chikibevchik/zabbixbyjenkins:web
                    '''
                }
            }
        }
    }
}
