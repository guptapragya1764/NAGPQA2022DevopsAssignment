currentBuild.displayName="NAGPDevTestops#"+currentBuild.number

pipeline {

agent any

tools{
        maven 'Maven'
    }
	
  triggers {
  pollSCM '*/2 * * * *'
  cron '* */12 * * *'
}


 environment{
        DEFUALT_MAIL_LIST = 'pragyagupta1792@gmail.com, cutubittu@gmail.com'
    }
		
stages {
  stage('Git Checkout') {
     steps{
     checkout scm
     }
  }

	stage("SonarQube Analysis"){
             steps{
             withSonarQubeEnv("Test_Sonar")
                 {
                 bat 'mvn sonar:sonar'
                 }
             }
         }
	stage("Quality Gate check status"){
		 steps{
           timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true, credentialsId: 'sonar-jenkin-token'
              }
          }
      }
	
	stage("Code build and test"){
		steps{
	              bat 'mvn clean test'
		}
	}

	
	stage("Publish to Artifactory"){
            steps{
                rtMavenDeployer(
                    id: 'deployer',
                    serverId: '547254@artifact',
                    releaseRepo: 'pragya-artifactory',
                    snapshotRepo: 'pragya-artifactory'
                )
                rtMavenRun(
                    pom: 'pom.xml',
                    goals: 'clean install',
                    deployerId: 'deployer'
                    )
                rtPublishBuildInfo(
                    serverId:'547254@artifact',
                )
            }        
        }
}
   
   post{
   always{
	    script {
	   if(env.DEFUALT_MAIL_LIST){
                    emailext subject: "${env.JOB_NAME} - BuildId#${env.BUILD_NUMBER} is ${currentBuild.currentResult}!", mimeType: 'text/html', 
                            to: "${env.DEFUALT_MAIL_LIST}", body: '${SCRIPT, template="groovy-html.template"}'
                }
	    }
		}
   }
}
