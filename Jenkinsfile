pipeline {
    agent {
        docker { 
            image 'alpine:3.6' 
            args '-u root:root'
		
        }
    }
    
    stages {
        
        stage('Install Dependencies') {
            steps {
                sh 'apk add --update curl zip wget tar libressl libressl-dev postgresql postgresql-dev redis openjdk8 maven openssh ca-certificates cyrus-sasl-dev bash'
            }
        }
        stage('init') {
            steps {
                    
		    sh '''	
    		        mkdir /run/postgresql
    		        chown postgres:postgres -R /run/postgresql/
    		        su - postgres -c "initdb -D /var/lib/postgresql/data"
    		        su - postgres -c "pg_ctl -D /var/lib/postgresql/data start"
    		        nohup redis-server >> redis.log &
    		        export JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk
    		        sleep 30s
    		        su - postgres -c "echo \"create user tactical with password 'password';\" | psql"
    		        su - postgres -c "echo 'create database tactical owner tactical;' | psql"
    		        su - postgres -c "echo \"create user tacticaltest with password 'testpassword';\" | psql"
    		        su - postgres -c "echo 'create database tacticaltest owner tacticaltest;' | psql"
    		    '''
            }
        }
    }
}
