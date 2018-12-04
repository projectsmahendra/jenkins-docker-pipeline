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
    		        su - postgres -c "echo \\"create user tactical with password 'password';\\" | psql"
    		        su - postgres -c "echo 'create database tactical owner tactical;' | psql"
    		        su - postgres -c "echo \\"create user tacticaltest with password 'testpassword';\\" | psql"
    		        su - postgres -c "echo 'create database tacticaltest owner tacticaltest;' | psql"
                        wget https://services.gradle.org/distributions/gradle-3.2.1-bin.zip
                        unzip gradle-3.2.1-bin.zip -d /usr/local
                        ln -s /usr/local/gradle-3.2.1 /usr/local/gradle
                        echo "export GRADLE_HOME=/usr/local/gradle" >> /etc/profile.d/gradle.sh
                        echo "export PATH=$GRADLE_HOME/bin:$PATH" >> /etc/profile.d/gradle.sh
                        source /etc/profile

                        mkdir -p artifacts
        
                        echo "starting stub server .........................................................."    
                        mkdir ./artifacts/stubServer
                        touch stub_server.log
                        wget --header "PRIVATE-TOKEN:Qbdcqb17xNo3gNgN8mtM" "https://gitlab.bcgtools.com/api/v3/projects/154/builds/artifacts/master/download?job=build" -O ./artifacts/stubServer/engine-stub-server.zip
                        unzip -q ./artifacts/stubServer/engine-stub-server.zip -d ./artifacts/stubServer/
                        unzip -q ./artifacts/stubServer/stub-server.zip -d ./artifacts/stubServer/
                        nohup sh ./artifacts/stubServer/service.sh start >> stub_server.log &
                        chmod -R 775 update_application_prop.sh
                            
                        echo "clean build ..................................................................."
                        ./gradlew clean build
    
                        cp build/libs/tactical-1.0-SNAPSHOT.jar .    
                        zip -r tactical.zip tactical-1.0-SNAPSHOT.jar Procfile run.sh
                        echo "tactical run .................................................................."    
                        nohup java -jar -Dlog_file=/tmp tactical-1.0-SNAPSHOT.jar --spring.profiles.active=dev --spring.config.location=config/application-dev.yml >> tactical.log &   
                        sleep 45s
            
                        echo "run tactical acceptance test ............................................................."
                        cd tactical-acceptance-test
                        ./gradlew clean cucumber >> tactical_acceptance_test.log 
                        
                        cd -
     
                        echo "run performance test..........................................................."
                        cd performance-test-suite
                        mvn clean install
                        cd -
    		    '''
            }
        }
    }
}
