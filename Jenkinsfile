def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]

pipeline {
    agent any
    
    
    environment{
        
        WORKSPACE = "${env.WORKSPACE}"
       
    }
    
    tools{
         maven 'localMaven'
         jdk 'localJdk'
    }

    stages {
        stage('Git Checkout') {
            steps {
                echo 'cloning the app code'
                git branch: 'main', url: 'https://github.com/poltop20/Jenkins-CICD-pipeline-project.git'
                
                    
            }
        }
        
        stage('Code Build') {
            steps {
                sh 'mvn clean package'
                sh 'java -version'
                
            }
            
            post { 
                success { 
                echo 'archiving ...!'
                archiveArtifacts artifacts: '**/*.war', followSymlinks: false
                }
                
            }    
    
        }
        stage('Unit Test'){
            steps {
              sh 'mvn test'
            }
       }
        stage('Integration Test'){
            steps {
              sh 'mvn verify -DskipUnitTests'
            }
        }
        stage ('Checkstyle Code Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        post {
            success {
                echo 'Generated Analysis Result'
            }
        }
    }
    
        stage('SonarQube Scan') {
          steps {
              
            withSonarQubeEnv('SonarQube') {
  
              
            sh """
                mvn sonar:sonar \
                -Dsonar.projectKey=JavaWebApp \
                -Dsonar.host.url=http://172.31.30.96:9000 \
                -Dsonar.login=7666789d850ee9862ed5872bd9fcd853c19c96b7
                """
                } 
            }
        }
        
        
        stage("Quality Gate"){
        
          steps{
           
            waitForQualityGate abortPipeline: true
           
        }
            
    }
    
    stage("artifact upload to Nexus"){
        
          steps{
           
            sh 'mvn clean deploy -DskipTests'
           
        }
            
    }
    
    
    
    
    stage('Deploy to DEV') {
      environment {
        HOSTS = "dev"
      }
      steps {
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
     }
    
    
    stage('Deploy to stage env') {
      environment {
        HOSTS = "stage"
      }
      steps {
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
     }
     
    stage('Approval') {
        steps{
            input('Do you want to proceed')
        }
    }
     
    stage('Deploy to Prod env') {
      environment {
        HOSTS = "prod"
      }
      steps {
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
    }
    
    }    
    
    post { 
        always { 
            echo 'I will always say Hello again!'
            slackSend channel: '#glorious-w-f-devops-alerts', color: COLOR_MAP[currentBuild.currentResult], message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }

        
}
    


