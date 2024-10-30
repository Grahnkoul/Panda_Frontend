def imageName ="192.168.44.44:8082/panda_docker/frontend"
def dockerTag=""
def dockerRegistry="https://192.168.44.44:8082"
def registryCredentials="artifactory"


pipeline 
{
    agent 
    {
        label 'agent'
    }
 
    environment 
    {
        PIP_BREAK_SYSTEM_PACKAGES = 1
        scannerHome = tool 'SonarQube'
    }
 
    stages 
    {
        stage('Get_Code') 
        {
            steps 
            {
                checkout scm
            }
        }
 
        stage('Unit_tests') 
        {
            steps 
            {
                sh "pip3 install -r requirements.txt"
                sh "python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml"
            }
        }
 
        stage('SonarQube') 
        {
            steps 
            {
                withSonarQubeEnv('SonarQube') 
                {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        } 
        
        stage('Build_Docker_Image') 
        {
            steps 
            {
                script 
                {
                   dockerTag = "RC-${env.BUILD_ID}"
                   customImage = docker.build("$imageName:$dockerTag")
                }
            }
        }   
        
        stage('Push_Image_Artifactory') 
        {
            steps 
            {
                script 
                {
                   docker.withRegistry("$dockerRegistry", "$registryCredentials")  
                   {
                     customImage.push()  
                     customImage.push('lastest')
                    }
                }
            }
        }   
        
        
    }
    
    post 
    {
        always 
		{
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
    }
    

}
