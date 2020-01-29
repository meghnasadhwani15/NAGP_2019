pipeline
{
	agent any
	environment
	{
		scannerHome = tool name: 'sonar_scanner_dotnet', type: 'hudson.plugins.sonar.MsBuildSQRunnerInstallation'  
	}
	options
   {
      timeout(time: 1, unit: 'HOURS')
      
      // Discard old builds after 5 days or 5 builds count.
      buildDiscarder(logRotator(daysToKeepStr: '5', numToKeepStr: '5'))
	  
	  //To avoid concurrent builds to avoid multiple checkouts
	  disableConcurrentBuilds()
   }
		 
	stages
	{
		stage ('checkout')
		{
			steps
			{
				echo  " ********** Clone starts ******************"
				checkout scm
			}
		}
		stage ('nuget')
		{
			steps
			{
				bat "dotnet restore"	 
			}
		}
		
		stage ('Start sonarqube analysis')
		{
			steps
			{
			  withSonarQubeEnv('Test_Sonar') {
              bat """ dotnet "${scannerHome}/SonarScanner.MSBuild.dll" begin /k:projectkey /n:$JOB_NAME /v:1.0 """
              }
			}
		}
		
		stage ('build')
		{
			steps
			{
				bat "dotnet build -c Release -o WebApplication4/app/build"
			}	
		}
		
		stage ('SonarQube Analysis end')
		{	
			steps
			{
				withSonarQubeEnv('Test_Sonar'){
				bat """dotnet "${scannerHome}/SonarScanner.MSBuild.dll" end"""
			   }
			}
		}
		
		stage ('Release Artifacts')
		{
			steps
	        {
	           bat "dotnet publish -c Release -o WebApplication4/app/publish"
	        }
		}
		
		stage ('Docker Image')
		{
			steps
		    {
		        bat returnStdout: true, script: """ docker build --no-cache -t "dotnetcoreapp_meghnasadhwani:${BUILD_NUMBER}" ."""
		    }
		}
		
		stage ('Stop Running container')
		{
	         steps
	        {
	            bat """docker ps -q --filter \"name=meghnasadhwani\" | grep -q . && (docker stop meghnasadhwani && docker rm -fv meghnasadhwani) || true """
	        }
		}
		stage ('Docker deployment')
		{
			steps
			{
			    bat """ docker run --name meghnasadhwani -d -p 5000:80 "dotnetcoreapp_meghnasadhwani:${BUILD_NUMBER}" """			}
		    }
	}

	 post {
			always 
			{
				echo "*********** Executing post tasks like Email notifications *****************"
			}
		}
}
