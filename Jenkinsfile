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
        stage("Git clone") {
            steps {
                sh '''
                cd /home/dik/gitr/
                git clone https://github.com/olegstepit/18_E_LEARN         
                '''
            }
        }    
        stage("Build") {
            steps {
                sh '''
                cd /home/dik/gitr/jenk/Postgres
                docker build -t oleg222/zabk:post .
                cd /home/dik/gitr/jenk/Zab-serv
                docker build -t oleg222/zabk:serv .
                cd /home/dik/gitr/jenk/Zab-web
                docker build -t oleg222/zabk:web .
                '''
            }
        } 
        stage("Postgres") {
            steps {
                sh '''
                docker network create zabbix-net
                docker run \
                --name zabbix-postgres \
                --network zabbix-net \
                -v /var/lib/zabbix/timezone:/etc/timezone \
                -v /var/lib/zabbix/localtime:/etc/localtime \
                -d oleg222/zabk:post
                '''
            }
        }
        stage("Zab-serv") {
            steps {
                sh '''
                docker run \
                --name zabbix-server \
                --network zabbix-net \
                -v /var/lib/zabbix/alertscripts:/usr/lib/zabbix/alertscripts \
                -v /var/lib/zabbix/timezone:/etc/timezone \
                -v /var/lib/zabbix/localtime:/etc/localtime \
                -p 10051:10051 \
                -d oleg222/zabk:serv
                '''
            }
        }
        stage("Zab-web") {
            steps {
                sh '''
                docker run \
                --name zabbix-web \
                -p 80:8080 \
                -p 443:8443 \
                --network zabbix-net \
                -v /var/lib/zabbix/timezone:/etc/timezone \
                -v /var/lib/zabbix/localtime:/etc/localtime \
                -e PHP_TZ="Europe/Kiev" \
                -d oleg222/zabk:web
                '''
            }
        }
        stage("docker login") {
            steps {
                echo " ============== docker login =================="
                withCredentials([usernamePassword(credentialsId: 'DockerHub-Credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh '''
                    docker login -u $USERNAME -p $PASSWORD
                    '''
                }
            }
        }
        stage("docker push") {
            steps {
                echo " ============== pushing image =================="
                sh '''
                docker push oleg222/zabk:post
                docker push oleg222/zabk:serv
                docker push oleg222/zabk:web
                '''
            }
        }
    }
}
