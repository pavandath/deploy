pipeline{
    agent any
    stages{
        stage ('BUILD'){
            steps{
                echo "********************Building the code**********************"
                sh 'rm -rf spring-petclinic'   
                sh 'git clone https://github.com/pavandath/spring-petclinic.git'
                dir ('spring-petclinic'){
                sh 'mvn clean package -DskipTests -Dcyclonedx.skip=true'
                stash name: 'build-jar', includes: 'target/*.jar'   
            }
            }
        }

        stage('CodeQuality'){
            steps{
                echo "**********RUNNING CODEQUALITY TEST***********"
                dir ('spring-petclinic'){
                sh '''
                mvn clean verify sonar:sonar \
                 -Dsonar.projectKey=pipeline \
                 -Dsonar.host.url=http://35.225.231.58:9000 \
                 -Dsonar.login=sqp_600718924b5f1fcc4e752140992988da1d31cef5\
                 -DskipTests \
                 -Dcyclonedx.skip=true 

                 '''
                }
            }
        }
        stage('DockerBuild'){
            agent {label 'docker-slave'}
            environment{
                DOCKER_CREDS = credentials('docker_creds')
            }
            steps{
                echo "************BUILDING DOCKER IMAGE************"
                sh 'mkdir -p stashed'
                dir('stashed') {
                unstash 'build-jar'
                }
                sh 'ls -l'
                writeFile file: 'Dockerfile',
                text: '''
                FROM eclipse-temurin:21-jdk-jammy
                WORKDIR /app
                COPY stashed/target/*.jar app.jar
                EXPOSE 8080
                CMD ["java", "-jar", "app.jar"] 
                '''
                sh ' docker build -t pavandath510/spring:v8 .'
                sh "docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}"
                sh ' docker push pavandath510/spring:v8'

            }

        }
        stage ('DockerDeploy'){
            agent {
                label 'docker-slave'
            }
                steps{
                    echo'*********DEPLOYING THE APPLICATION*******************'
                    sh ' docker pull pavandath510/spring:v8'
                    sh ' docker run -d --name deployfinaldeevv8 -p 8600:8080 pavandath510/spring:v8'
                }
            }
    }
     

}
