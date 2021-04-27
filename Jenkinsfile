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
        sh "sudo docker run -p 3000:3000 -p 10000:10000 '$ACR_SERVER'/simple-spring-app"
        
    }
}


node("build-server"){
    
    dir('FunctionalTests'){
        
        stage('Get Functional Test Scripts'){      
            
            git 'https://github.com/devopsevd/azure-jenkins-selenium-testscript.git'
            
        }
        stage('Run Tests') {
            
            sh "'${mvnHome}/bin/mvn' -Dgrid.server.url=http://10.1.1.4:4444/wd/hub clean test "
        }
        stage('Functional Test Results') {
            junit '**/target/surefire-reports/TEST-*.xml'
        }
    }

}

//stage name:'Shutdown staging'
//    node {
                
//        dir('BuildQuality'){
//        sh 'sudo docker-compose stop'
//    }
                
//}
        

