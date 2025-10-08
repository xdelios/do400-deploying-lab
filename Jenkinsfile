pipeline{
    agent{
        node{
            label 'maven'
        }
    }
    environment { QUAY = credentials('QUAY_USER') }
    stages{
        stage('Test'){
            steps{
                sh './mvnw verify'
            }
        }

        stage('Build and push'){
            
            steps{
                sh ' ./mvnw quarkus:add-extension -Dextensions="container-image-jib"'
                sh '''
                    ./mvnw package -DskipTests \
                        -Dquarkus.jib.base-jvm-image=quay.io/redhattraining/do400-java-alpine-openjdk11-jre:latest \
                        -Dquarkus.container-image.build=true \
                        -Dquarkus.container-image.registry=quay.io \
                        -Dquarkus.container-image.group=$QUAY_USR \
                        -Dquarkus.container-image.name=do400-deploying-lab \
                        -Dquarkus.container-image.username=$QUAY_USR \
                        -Dquarkus.container-image.password="$QUAY_PSW" \
                        -Dquarkus.container-image.tag=build-${BUILD_NUMBER} \
                        -Dquarkus.container-image.additional-tags=latest \
                        -Dquarkus.container-image.push=true
                '''
            }
        }

        stage('Deploy to Test'){
            steps{
                sh """
                    oc set image deployment home-automation \
                    home-automation=quay.io/${QUAY_USR}/do400-deploying-lab:build-${BUILD_NUMBER} \
                    -n rplvvx-deploying-lab-test --record
                """
            }
        }
    }


}