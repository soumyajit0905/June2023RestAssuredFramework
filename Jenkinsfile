pipeline 
{
    agent any
    
    tools{
    	maven 'maven'
        }
        
    environment {
        BUILD_NUMBER = "${BUILD_NUMBER}"
        DOCKER_IMAGE_NAME = "apitesting-new:${BUILD_NUMBER}"
        DOCKER_REGISTRY = "naveenkhunteta" // Change this to your Docker Hub username
    }
    

    stages 
    {
        stage('Build') 
        {
            steps
            {
                 git 'https://github.com/jglick/simple-maven-project-with-tests.git'
                 bat "mvn -Dmaven.test.failure.ignore=true clean package"
            }
            post 
            {
                success
                {
                    junit '**/target/surefire-reports/TEST-*.xml'
                    archiveArtifacts 'target/*.jar'
                }
            }
        }

        
        
        stage("Deploy to QA"){
            steps{
                echo("deploy to qa done")
            }
        }
        
        
        stage('Docker Build') {
            steps {
                script {
                    def dockerBuildCommand = """
                        docker build -t ${DOCKER_IMAGE_NAME} .
                    """
                    bat(script: dockerBuildCommand)
                }
            }
        }
        
        stage('Tag and Push Docker Image') {
            steps {
                script {
                    def dockerTagCommand = """
                        docker tag ${DOCKER_IMAGE_NAME} ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}
                    """
                    bat(script: dockerTagCommand)
                    
                    def dockerPushCommand = """
                        docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}
                    """
                    bat(script: dockerPushCommand)
                }
            }
        }
        
             
             
                
                
      stage('Run Docker Image with Regression Tests') {
    steps {
        script {
            def suiteXmlFilePath = 'src/test/resources/testrunners/testng_regression.xml'
            def dockerImage = "${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}" // Use the tagged image name

            def dockerRunCommand = """
                docker run --name apitesting${BUILD_NUMBER} \
                -v "${WORKSPACE}/reports:/app/reports" \
                ${dockerImage} \
                /bin/bash -c "mvn test -Dsurefire.suiteXmlFiles=${suiteXmlFilePath}"
            """
            
            def exitCode = bat(script: dockerRunCommand, returnStatus: true)
            
            if (exitCode != 0) {
                currentBuild.result = 'FAILURE'
            }

            def dockerStartCommand = "docker start apitesting${BUILD_NUMBER}"
            def dockerCopyCommand = "docker cp apitesting${BUILD_NUMBER}:/app/reports/APIExecutionReport.html ${WORKSPACE}/reports"
            def dockerRemoveCommand = "docker rm -f apitesting${BUILD_NUMBER}"
            
            bat script: dockerStartCommand, returnStatus: true
            bat script: dockerCopyCommand, returnStatus: true
            bat script: dockerRemoveCommand, returnStatus: true
        }
    }
}




		
		stage('Publish Regression Extent Report'){
            steps{
                     publishHTML([allowMissing: false,
                                  alwaysLinkToLastBuild: false, 
                                  keepAll: false, 
                                  reportDir: 'reports', 
                                  reportFiles: 'APIExecutionReport.html', 
                                  reportName: 'API HTML Regression Extent Report', 
                                  reportTitles: ''])
            }
        }
        
        
         

         
    }
}