//  Pipeline //
pipeline {
    agent any
    
    stages {

        stage('Build') {
            steps{
                script{
                     sh 'cd webdemo && ./gradlew build'
                     if(!continueBuild){
                        echo '============ssssssss================'
                        currentBuild.result = 'ABORTED'
                        error("stopping early")
                     }else{
                         echo "为什么没有生效"
                     }
                }
            }
        }

        
        
        stage('Test') {
            steps{
                script{
                    echo 'Testing..'
                    sh 'cd webdemo && ./gradlew test'
                    
                    if(currentBuild.result== null){
                        echo '============================'
                        currentBuild.result = 'ABORTED'
                        error("stopping early")
                    }
                    
                    echo '+++++++++++++++++++++++++'
                }
            }
        }

        stage('SonarQube analysis') {
            steps{
                echo "starting codeAnalyze with SonarQube......"
                script{
                    def sonarqubeScannerHome = tool name:'SonarScannerTest'
                    withSonarQubeEnv('SonarSeverTest') {
                        sh "${sonarqubeScannerHome}/bin/sonar-scanner"
                    }
            
                    timeout(4) { 
                   //利用sonar webhook功能通知pipeline代码检测结果，未通过质量阈，pipeline将会fail
                   def qg = waitForQualityGate() 
                       if (qg.status != 'OK') {
                           error "It doesn't pass Sonarqube scanner gate setting，Please fix it！failure: ${qg.status}"
                       }
                    }
                } 
            }
        }
        
        stage('Deploy') {
            steps{
                 echo 'Deploying..' 
                sh """
                    set -e
                    ssh hbao@10.209.22.46 'bash -s' < checktomcatstatus.sh
                    cd /var/jenkins_home/workspace/TestForPipeline/webdemo/build/libs
                    scp webdemo.war hbao@10.209.22.46:/Users/hbao/Downloads/apache-tomcat-7.0.82/webapps
                    ssh hbao@10.209.22.46 '
                        cd /Users/hbao/Downloads/apache-tomcat-7.0.82/bin
                        ./startup.sh
                    '
                """ 
            }
        }
        
    }
}



