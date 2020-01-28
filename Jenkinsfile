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
		
		stage ('build')
		{
			steps
			{
				bat "dotnet build -c Release -o WebApplication4/app/build"
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
	           bat """
                   set ContainerID=/$(docker ps | grep 5000 | cut -d " " -f 1)
                   if [  %ContainerID% ]
                   then
                      docker stop %ContainerID%
                      docker rm -f %ContainerID%
                   
                """
	       }
		}
		stage ('Docker deployment')
		{
			steps
			{
			    bat """ docker run --name dotnetcoreapp_meghnasadhwani -d -p 5000:80 "dotnetcoreapp_meghnasadhwani:${BUILD_NUMBER}" """			}
		}
	}

	 post {
			always 
			{
				echo "*********** Executing post tasks like Email notifications *****************"
			}
		}
}
