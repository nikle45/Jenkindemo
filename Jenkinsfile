pipeline{
    agent any 
    tools{
        jdk 'jdk11'
        maven 'maven3'
    }
    environment{
        SCANNER_HOME=tool 'sonar-scanner'
    }
    
    stages{
        
        stage("Git checkout"){
            steps{
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/nikle45/Jenkindemo.git'
            }
        } 
        
        stage("compile"){
            steps{
                sh "mvn clean compile -Dcheckstyle.skip"
            }
        }
        
        stage("Test"){
            steps{
                sh "mvn test -Dcheckstyle.skip"
            }
        }
        
        stage("Sonar Analysis"){
            steps{
                withSonarQubeEnv('sonar-server') {
                sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petclinic \
                -Dsonar.java.binaries=. \
                -Dsonar.projectKey=Petclinic '''
                }
            }
        }
        
         stage("OWASP Dependency check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format HTML', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.html'
            }
        }
        
         stage("Build"){
            steps{
                sh "mvn clean install -Dcheckstyle.skip"
            }
        }
        
         stage("Docker Build and push "){
            steps{
                script{
                    withDockerRegistry(credentialsId: '6cad6fd2-173d-427e-b2b2-64fe36aca811', toolName: 'docker') {
                    
                    sh "docker build -t image1 ."
                    sh "docker tag image1 ghalge/pet-clinic123_$BUILD_NUMBER:latest"
                    sh "docker push ghalge/pet-clinic123_$BUILD_NUMBER:latest"
                    }
                }
            }
        }
        
        stage("deploy"){
            steps{
                sh "java -Dserver.port=8888 -jar target/spring-petclinic-2.3.1.BUILD-SNAPSHOT.jar"
            }
        }
        
    }
}
