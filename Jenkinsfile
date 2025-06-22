pipeline{
    agent any
    stages{
        stage ('BUILD'){
            steps{
                echo "********************Building the code**********************"
                sh 'git clone https://github.com/pavandath/spring-petclinic.git'
                dir ('spring-petclinic'){
                sh 'mvn clean package'
                stash name: 'build-jar', includes: 'target/*.jar'   
            }
            }
        }

        stage('CodeQuality'){
            steps{
                echo "**********RUNNING CODEQUALITY TEST***********"
                sh '''
                mvn clean verify sonar:sonar \
                 -Dsonar.projectKey=pipeline \
                 -Dsonar.host.url=http://35.225.231.58:9000 \
                 -Dsonar.login=sqp_4c8f37cca02dc15840dd56a2c455b4dba4cae502
                 '''
            }
        }
        stage('DockerBuild'){
            agent {label 'docker-slave'}
            environment{
                DOCKER_CREDS = credentials('docker_creds')
            }
            steps{
                echo "************BUILDING DOCKER IMAGE************"
                unstash 'build-jar'
                writeFile file: 'Dockerfile',
                text: '''
                FROM openjdk:17-jdk-slim
                WORKDIR /app
                COPY target/*.jar app.jar
                CMD ['java' , '-jar ', 'app.jar'] 
                '''
                sh 'docker build -t pavandath510/spring:v1 .'
                sh 'docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PWS} '
                sh 'docker push pavandath510/spring:v1'

            }

        }
        stage ('DockerDeploy'){
                steps{
                    sh 'docker pull pavandath510/spring:v1'
                    sh 'docker run -d --name deployapp -p 8000:8080 pavandath510/springv1'
                }
            }
    }
     

}
