node("build-server"){

    checkout scm
    def mvnHome
    dir('BuildQuality'){
        stage('Preparation'){
                        
            git 'https://github.com/devopsevd/simple-spring-azure.git'
            mvnHome = tool 'Maven'
        }

        stage('Build') {
            // Run the maven build
            if (isUnix()) {
                sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
            } 
        }
        
        stage('SonarQube Analysis') { 
           // def mvnHome
            //mvnHome = tool 'Maven'
            withSonarQubeEnv('Sonar') { 
                if (isUnix()) {
                    sh "'${mvnHome}/bin/mvn' org.sonarsource.scanner.maven:sonar-maven-plugin:3.3.0.603:sonar -f pom.xml "+ 
                    " -Dsonar.projectKey=org.sonarqube:java-sonar " +
                    " -Dsonar.projectKey=org.sonarqube:java-sonar " +
                    " -Dsonar.projectName='Java :: Simple Spring Project' " +
                    " -Dsonar.projectVersion=1.0 " +
                    " -Dsonar.language=java " +
                    " -Dsonar.sources=. " +
                    " -Dsonar.tests=. " +
                    " -Dsonar.test.inclusions='**/*Test*/**' " +
                    " -Dsonar.exclusions='**/*Test*/**' "
                }     
            }        
        }

        stage('Unit Test Results') {
            junit '**/target/surefire-reports/TEST-*.xml'
        }
        
        stage('Package') {
             withCredentials([usernamePassword(
                                credentialsId: 'acr',
                                passwordVariable: 'PASSWORD',
                                    usernameVariable: 'USER')]) {
                 
                                sh "sudo docker login -u '$USER' -p '$PASSWORD' '$ACR_SERVER'"
                        }
            sh 'sudo docker-compose build'
            sh "sudo docker tag simple-spring-app '$ACR_SERVER'/simple-spring-app"
            sh "sudo docker push '$ACR_SERVER'/simple-spring-app"
        }

    }
}

node("test-server"){
    stage('Deploy to Test Env'){
          withCredentials([usernamePassword(
                       credentialsId: 'acr',
                       passwordVariable: 'PASSWORD',
                       usernameVariable: 'USER')]) {
                 
              sh "sudo docker login -u '$USER' -p '$PASSWORD' '$ACR_SERVER'"
           }
        sh "sudo docker ps -q --filter ancestor='$ACR_SERVER'/simple-spring-app | xargs -r sudo docker stop"
        sh "sudo docker ps -a -q --filter ancestor='$ACR_SERVER'/simple-spring-app | xargs -r sudo docker rm"
        //sh "sudo docker rmi '$ACR_SERVER'/simple-spring-app"
        sh "sudo docker run -d -p 3000:3000 -p 10000:10000 '$ACR_SERVER'/simple-spring-app"
        
    }
}


node("build-server"){
    def mvnHome
    dir('FunctionalTests'){
        
        stage('Get Functional Test Scripts'){      
            
            git 'https://github.com/devopsevd/azure-jenkins-selenium-testscript.git'
            mvnHome = tool 'Maven'
        }
        stage('Run Tests') {
            
            sh "'${mvnHome}/bin/mvn' -Dgrid.server.url=http://10.1.1.4:4444/wd/hub clean test "
        }

    }

}

node("stage-server"){
    stage('Deploy to Stage Env'){
          withCredentials([usernamePassword(
                       credentialsId: 'acr',
                       passwordVariable: 'PASSWORD',
                       usernameVariable: 'USER')]) {
                 
              sh "sudo docker login -u '$USER' -p '$PASSWORD' '$ACR_SERVER'"
           }
        sh "sudo docker ps -q --filter ancestor='$ACR_SERVER'/simple-spring-app | xargs -r sudo docker stop"
        sh "sudo docker ps -a -q --filter ancestor='$ACR_SERVER'/simple-spring-app | xargs -r sudo docker rm"
        //sh "sudo docker rmi '$ACR_SERVER'/simple-spring-app"
        sh "sudo docker run -d -p 3000:3000 -p 10000:10000 '$ACR_SERVER'/simple-spring-app"
        
    }
}

node("build-server"){
    dir('Performance Test'){        
        stage('Performance tests'){                  
            git 'https://github.com/devopsevd/terraform-jmeter-azure.git'
            sh "./exec-jmeter.sh 3"
            perfReport compareBuildPrevious: true, percentiles: '0,50,90,100', sourceDataFiles: '**/results/**/results'           
        }
    }
}

node("build-server"){
    dir('Security Test'){        
        stage('Security tests'){                  
            git 'https://github.com/devopsevd/terraform-zap-azure.git'
            sh "sudo docker run --rm -t owasp/zap2docker-stable zap-baseline.py -t http://10.1.0.6:10000/ -u https://raw.githubusercontent.com/devopsevd/terraform-zap-azure/master/config"
                     
        }
    }
}

 

