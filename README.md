# MYSQL_Queries_Run_Using_Jenkins
In this repository i will describe the way to run MYSQL Queries through on several DB servers code written in Groovy

# Problem Statement
When we have a team of developers there is always a need of developers to run various MYSQL queries on DB or they want to update, Delete or insert data in DB for their task but as per organization rules sometimes we can give them server access of they have limitted access to server or we cant give production server access to developers. If we will go with MYSQL workbench or other tools that are availabe we can't monitor which developer run which query and what is the status of their queries etc.Also we can restrict to run only few queries like they can run only SELECT queries or UPDATE and SELECT etc.

# Solution
With the help of jenkins Pipeline we can solve this problem by Running MYSQL queries through jenkins.

# Requirements
# MYSQL
# Jenkins

#######################################################################################################################################
# Steps to create a Jenkins Job through which we can run MYSQL queries on various servers
(Activity done on DB server)
# Step 1 : 
First you need to create a remote user in DB server so that you can connect it through your Jenkins server
Login to DB server and run below commands

  CREATE USER 'jenkins'@'192.168.100.212' IDENTIFIED BY 'password';
  
  ALTER USER 'jenkins'@'192.168.100.211' IDENTIFIED WITH mysql_native_password BY 'password'; (IF using MYSQL 8 then ALter user to MYSQL Native password)
  
  GRANT SELECT, UPDATE ON DB.* TO 'jenkins'@'192.168.100.211'; (you can choose what permissions you want to give to jenkins users)
  
  flush privileges;
  
  # Step 2 
  Open mysql port to Jenkins server as if DB server is on cloud open port 3306 to Jenkins server so that we can connect, check firewall and open port for Jenkins server. 
  
  (Activities on Jenkins server)
  # Step 3
  check connectivity with the help of below command
  telnet DB_server_IP 3306 (run this command from Jenkins server in command prompt if you are able to connect then done otherwise troubleshoot it might be due to firewall or security groups in cloud)
 
   # Step 4
   Login to Jenkins Dashboard and create a pipeline job and paste Below Groovy script
   
   pipeline {
    agent any
    parameters { 
	           choice(name: 'OEM', choices: [ 'Nokia', 'Samsung','MI','Vivo' ], description: 'Please Select OEM to run DB Query')
			   string(name: 'Enter_DB_Query', defaultValue: ' ', description: 'Please enter your DB query')
			   string(name: 'Enter_DB_Password', defaultValue: ' ', description: 'Please enter your DB Password', trim: true)
                       }
	     stages {
         stage('Initial'){
		        steps {
			     echo "You have selected ${params.OEM} to run DB query"
                 echo "You have entered ${params.Enter_DB_Query} to run"
                         }
		                      }
		  stage('Query Run'){
			steps {
				script{
				         if (params.OEM == 'Nokia'){
						 sh 'mysql -h DB_Server_IP -u jenkins -p$Enter_DB_Password -D nokia -e "$Enter_DB_Query;" '
						                                                   }
						 else if (params.OEM == 'Samsung'){
						 sh 'mysql -h DB_Server_IP -u jenkins -p$Enter_DB_Password -D samsung -e "$Enter_DB_Query;" '                                                         
																					 }
						 else if (params.OEM == 'MI'){
						 sh 'mysql -h DB_Server_IP -u jenkins -p$Enter_DB_Password -D mi -e "$Enter_DB_Query;" '
                                                                                   }
                         else if (params.OEM == 'Vivo'){
						 sh 'mysql -h DB_Server_IP -u jenkins -p$Enter_DB_Password -D vivo -e "$Enter_DB_Query;" '
                                                                                   }
																													   
						 else {
						    echo "Please Select Right OEM"        
								  }
					         }                                                 
						 }
		     	    }
	        }
	    post {
        always {
            // With the help of this segment you will recieve an Email with the status of build, which user run this query, what query is run with complete build log
			echo 'Sending Email!'
			wrap([$class: 'BuildUser']) {
                emailext attachLog: true, body: "${currentBuild.currentResult} : Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n DB Query Run : ${params.Enter_DB_Query}\n on ${params.OEM} \n By : ${BUILD_USER}",
                recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
                subject: "Jenkins Build Trigger By ${BUILD_USER} Job ${env.JOB_NAME} : ${currentBuild.currentResult}",
                to : 'farrukhfareed2009@gmail.com'
            }
        }
    }
        }

# Step 5 
Configure STMP details for sending emails from jenkins server. Also for the very first time when you run this build it might be failed or blank but this is beacuse when you save with this code you saw only build button but our job is parametrized after running first jon it will set all the parameters and if again you run this job you will saw a drop down menu for OEM selection and portions now it will work .

Note : if any one has difficulty and any query regarding this feel free to contact me on my email : farrukhfareed2009@gmail.com
  
  
