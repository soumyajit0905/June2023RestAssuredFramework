pipeline 
{
    agent any
    
    tools{
    	maven 'maven'
        }

    stages 
    {
        stage('Build') 
        {
            steps
            {
                 git 'https://github.com/jglick/simple-maven-project-with-tests.git'
                 sh "mvn -Dmaven.test.failure.ignore=true clean package"
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
                
        stage('Run Docker Image with Tests') {
    steps {
        script {
            sh "docker run --name apitesting -e MAVEN_OPTS='-Dsurefire.suiteXmlFiles=src/test/resources/testrunners/testng_regression.xml' naveenkhunteta/my-maven-api:latest"
       	    sh "docker start apitesting"
       	    sh "sleep 60"
       	    sh "docker cp apitesting:/app/reports/APIExecutionReport.html ${WORKSPACE}"
       	    
       	    sh "docker rm -f apitesting"
       			
       			 }
    		}
		}

         
    }
}