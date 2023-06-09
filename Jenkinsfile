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
        stage("Build") {
            steps {
                sh '''
                cd /var/lib/jenkins/workspace/zabbix/Postgres
                docker build -t darkne24/zabk:post .
                cd /var/lib/jenkins/workspace/zabbix/Zab-serv
                docker build -t darkne24/zabk:serv .
                cd /var/lib/jenkins/workspace/zabbix/Zab-web
                docker build -t darkne24/zabk:web .
                '''
            }
        } 
        stage("Postgres") {
            steps {
                sh '''
                docker run \
                --name zabbix-postgres \
                --network zabbix-net \
                -v /var/lib/zabbix/timezone:/etc/timezone \
                -v /var/lib/zabbix/localtime:/etc/localtime \
                -d darkne24/zabk:post
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
                -d darkne24/zabk:serv
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
                -d darkne24/zabk:web
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
                docker push darkne24/zabk:post
                docker push darkne24/zabk:serv
                docker push darkne24/zabk:web
                '''
            }
        }
    }
}
