def label = "mypod-${UUID.randomUUID().toString()}"
def serviceaccount = "jenkins-admin"

podTemplate(label: label, serviceAccount: serviceaccount, containers: [
	containerTemplate(name: 'maven', image: 'localhost:32121/root/docker_registry/aiindevops.azurecr.io/maven:3.3.9-jdk-8-alpine', ttyEnabled: true, command: 'cat',resourceRequestCpu: '150m',resourceLimitCpu: '4000m',resourceRequestMemory: '100Mi',resourceLimitMemory: '7000Mi'),
	containerTemplate(name: 'git-secrets', image: 'localhost:32121/root/docker_registry/aiindevops.azurecr.io/git-secrets:0.1', ttyEnabled: true, alwaysPullImage: true, command: 'cat'),
	containerTemplate(name: 'jq', image: 'localhost:32121/root/docker_registry/aiindevops.azurecr.io/jq', ttyEnabled: true, command: 'cat'),
	containerTemplate(name: 'jmeter', image: 'localhost:32121/root/docker_registry/aiindevops.azurecr.io/jmeter', ttyEnabled: true, command: 'cat'),
	containerTemplate(name: 'clair-scanner', image: 'localhost:32121/root/docker_registry/aiindevops.azurecr.io/clair-scanner:0.1', ttyEnabled: true, alwaysPullImage: true, command: 'cat', ports: [portMapping(name: 'clair-scanner', containerPort: '9279')],envVars: [
            envVar(key: 'DOCKER_SOCK', value: "$DOCKER_SOCK"),envVar(key: 'DOCKER_HOST', value: "$DOCKER_HOST")],
					volumes: [hostPathVolume(hostPath: "$DOCKER_SOCK", mountPath: "$DOCKER_SOCK")]),
	containerTemplate(name: 'kubeaudit', image: 'localhost:32121/root/docker_registry/aiindevops.azurecr.io/kube-audit:0.1', ttyEnabled: true, alwaysPullImage: true, command: 'cat'),
	containerTemplate(name: 'kubectl', image: 'localhost:32121/root/docker_registry/aiindevops.azurecr.io/docker-kubectl:19.03-alpine', ttyEnabled: true, command: 'cat'),
	containerTemplate(name: 'ansible', image: 'ansibleplaybookbundle/s2i-apb', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'docker', image: 'localhost:32121/root/docker_registry/aiindevops.azurecr.io/docker:1.13', ttyEnabled: true, command: 'cat',envVars: [
            envVar(key: 'DOCKER_SOCK', value: "$DOCKER_SOCK"),envVar(key: 'DOCKER_HOST', value: "$DOCKER_HOST")])],
			volumes: [hostPathVolume(hostPath: "$DOCKER_SOCK", mountPath: "$DOCKER_SOCK")],
		imagePullSecrets: ['gcrcred']
) {
    node(label) {  
        def GIT_URL= 'http://gitlab.ethan.svc.cluster.local:8084/gitlab/devopsuser1/module4_continuousdelivery.git'
		def GIT_CREDENTIAL_ID ='gitlab'
		def GIT_BRANCH='master'
        		
		/***COMPONENT_KEY value should be same as what is given as ProjectName in in sonar.properies file ***/		
		def COMPONENT_KEY='carts';
		def rootDir = pwd()	
		def SONAR_UI='http://sonar.ethan.svc.cluster.local:9001/sonar/api/measures/component?metricKeys=';
		
		/**** provide Sonar metrickeys which needs to be published to Jenkins console ***/
		String metricKeys = "coverage,code_smells,bugs,vulnerabilities,sqale_index,tests,ncloc,quality_gate_details,duplicated_lines_density,major_violations,minor_violations,critical_violations,blocker_violations,security_rating,complexity,violations,open_issues,test_success_density,test_errors,test_execution_time,security_remediation_effort,uncovered_conditions,classes,functions,line_coverage,sqale_rating,sqale_debt_ratio,reliability_remediation_effort,coverage,code_smells,bugs,vulnerabilities,sqale_index,tests,ncloc,quality_gate_details,duplicated_lines,cognitive_complexity";
		
		/*** Below variables used in the sonar maven configuration ***/
		def SONAR_SCANNER='org.sonarsource.scanner.maven'
		def SONAR_PLUGIN='sonar-maven-plugin:3.2'
		def SONAR_HOST_URL='http://sonar.ethan.svc.cluster.local:9001/sonar'
		
		/***  DOCKER_HUB_REPO_URL is the URL of docker hub ***/
		
		def GCR_HUB_ACCOUNT = 'localhost:32121'
		def GCR_HUB_ACCOUNT_NAME = 'root'
		def GCR_HUB_REPO_NAME='docker_registry'
		def DOCKER_IMAGE_NAME = 'java'
		def ACR_HUB_ACCOUNT = 'aiindevops.azurecr.io'
        def IMAGE_TAG = '1.7.6'

        def  JOBNAME = "${JOB_NAME.split('/')[1]}"
				
		def K8S_DEPLOYMENT_NAME = 'pet-clinic'
                        
     try {
		
		stage('Git Checkout') {
			
			git branch: GIT_BRANCH, url: GIT_URL,credentialsId: GIT_CREDENTIAL_ID
            def function = load "${WORKSPACE}/JenkinsFunctions_Java.groovy"
			def Nap = load "${WORKSPACE}/git_scan_nonallowed.groovy"
			def Ap = load "${WORKSPACE}/git_scan_allowed.groovy"
			
			// Below two lines are to publish last commited user name and email into jenkins console logs
            sh 'GIT_NAME=$(git --no-pager show -s --format=\'%an\' $GIT_COMMIT)'
            sh 'GIT_EMAIL=$(git --no-pager show -s --format=\'%ae\' $GIT_COMMIT)'
			
        stage('Git Secret') {
		    container('git-secrets') {
		        echo "${IMAGE_TAG}"
				Nap.nonAllowedPattern()
				Ap.AllowedPattern()	
				sh 'git secrets --scan'
			}
		}
		
		committerEmail = sh (
        script: 'git --no-pager show -s --format=\'%ae\'',
        returnStdout: true
        ).trim()
        echo "$committerEmail"
        sh '''
        committerEmail='''+ committerEmail +'''
        echo "committerEmail is $committerEmail"
        '''
        sh '''
        GIT_NAME=$(git --no-pager show -s --format=\'%an\' $GIT_COMMIT)
        echo "GIT_NAME is ${GIT_NAME}"
        '''

        wrap([$class: 'BuildUser'])
        {
            sh 'echo ${BUILD_USER}'
			sh '''
			committerEmail='''+ committerEmail +'''
			BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
			echo ${BUILDUSER}
		    sed -i "s/pet-clinic/${BUILDUSER}/g" tomcat.yaml
		    sed -i "s/pet-clinic/${BUILDUSER}/g" tomcat-svc.yaml
            sed -i "s/pet-clinic/${BUILDUSER}/g" namespace.yaml
            sed -i "s/pet-clinic/${BUILDUSER}/g" sonar-project.properties
            sed -i "s/deployenvnum/${BUILD_NUMBER}${BUILDUSER}${deploy_env}/g" tomcat.yaml
		    '''
        }
		
		stage('Application_Build') {
			container('maven') {
				
				function.buildMethod()
				
			}
			
		} 
		
		
            
		stage('Application_UnitTest') {
			container('maven') {
				function.testMethod()
			}
		}  
                           
                             
		 stage('Static_Code_Analysis') {
			withCredentials([usernamePassword(credentialsId: 'SONAR', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]){
			//withSonarQubeEnv('SonarQube') {
			container('maven') {
			sh ("curl -u ${USERNAME}:${PASSWORD} -k http://sonar.ethan.svc.cluster.local:9001/sonar/api/qualitygates/show?name=java_QG > qualitygate.json")
			sh 'cat qualitygate.json'
			}
			container('jq') {
			sh '''
            committerEmail='''+ committerEmail +'''
			BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
            export SONAR_PROJECT_KEY="$(BUILDUSER)"
            export SONAR_QG_NAME='java_QG'
			apt update
			apt -y install curl
			echo $SONAR_PROJECT_NAME
			echo $SONAR_PROJECT_KEY
			echo $SONAR_QG_NAME
			echo "creating sonarqube gateway"
			curl -u ${USERNAME}:${PASSWORD} -k -X POST "http://sonar.ethan.svc.cluster.local:9001/sonar/api/qualitygates/create?name=$SONAR_QG_NAME"
			curl -u ${USERNAME}:${PASSWORD} -k "http://sonar.ethan.svc.cluster.local:9001/sonar/api/qualitygates/show?name=$SONAR_QG_NAME" > qualitygate.json
		    QG_ID=$(cat qualitygate.json | jq -r ".id")
		    curl -u ${USERNAME}:${PASSWORD} -k -X POST "http://sonar.ethan.svc.cluster.local:9001/sonar/api/qualitygates/create_condition?gateId=$QG_ID&metric=coverage&op=LT&error=90"
			curl -u ${USERNAME}:${PASSWORD} -k -X POST "http://sonar.ethan.svc.cluster.local:9001/sonar/api/qualitygates/create_condition?gateId=$QG_ID&metric=duplicated_lines_density&op=GT&error=1"
			export SONAR_PROJECT_NAME="$(BUILDUSER)"
			curl -u ${USERNAME}:${PASSWORD} -k -X POST "http://sonar.ethan.svc.cluster.local:9001/sonar/api/projects/create?project=$SONAR_PROJECT_KEY&name=$SONAR_PROJECT_NAME"
			curl -u ${USERNAME}:${PASSWORD} -k "http://sonar.ethan.svc.cluster.local:9001/sonar/api/projects/search?" > project.json
			ls -lrta
			cat qualitygate.json
			cat project.json
			PROJECT_ID=$(cat project.json | jq -r \'.components[] | select(.name=="'$SONAR_PROJECT_NAME'") | .id\')
			echo $QG_ID
			echo $PROJECT_ID
			curl -u ${USERNAME}:${PASSWORD} -k -X POST "http://sonar.ethan.svc.cluster.local:9001/sonar/api/qualitygates/select?gateId=$QG_ID&projectKey=$SONAR_PROJECT_KEY"
			echo "Updating sonarqube gateway"
			curl -u ${USERNAME}:${PASSWORD} -k "http://sonar.ethan.svc.cluster.local:9001/sonar/api/qualitygates/show?name=$SONAR_QG_NAME" > qualitygate.json
			 cat qualitygate.json | jq -r ".conditions[].id" > cgid.txt
            cgid1=$(head -n 1 cgid.txt)
            echo $cgid1
            curl -u ${USERNAME}:${PASSWORD} -k -X POST "http://sonar.ethan.svc.cluster.local:9001/sonar/api/qualitygates/update_condition?gateId=$QG_ID&id=$cgid1&metric=coverage&op=LT&error=0"
            cgid2=$(sed -n '2p' cgid.txt)
            echo $cgid2
            curl -u ${USERNAME}:${PASSWORD} -k -X POST "http://sonar.ethan.svc.cluster.local:9001/sonar/api/qualitygates/update_condition?gateId=$QG_ID&id=$cgid2&metric=duplicated_lines_density&op=GT&error=100"
			'''
          }
			
		container('maven') {
			sh '''
			apk update
			apk add nodejs
			'''
				withSonarQubeEnv('SonarQube') {
                println('Sonar Method enter');
				function.sonarMethod()
                echo "Access the SonarQube URL from the Platform Dashboard tile"
				sh 'sleep 30'
               }
			   
			   timeout(time: 1, unit: 'HOURS') {
                                def qg = waitForQualityGate()
                                if (qg.status != 'OK') {
                                    error "Pipeline aborted due to quality gate failure: ${qg.status}"
                                }
                            }
		}
             }
			}		
        
		/* Below stage is to publish tools logs to Jenkins console
		stage('Publish') {
			container ('maven') {
				String[] metricKeyList = metricKeys.split(",");
                     for(String key : metricKeyList){
                     String var1=SONAR_UI+key+'&componentKey='+COMPONENT_KEY;
                     sh('curl '+'"'+var1+'"');  
                  } 
			}
		} */
		
		/*stage('Create Docker Image'){*/
			container('docker'){
				
				sh '''
                committerEmail='''+ committerEmail +'''
                BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
                docker build -t localhost:32121/root/docker_registry/java:${BUILD_NUMBER}${BUILDUSER}${deploy_env} .
                '''
			}
		/*}*/
		
	
		
	/*	stage('Create Jenkinslave Service') {*/
            container('kubectl') {
            echo 'Deploying....'
            
            sh """
			echo ${label}
			cat ${WORKSPACE}/clair-scanner.yaml | sed "s/{{parm}}/${label}/g" | kubectl apply -f -
			"""			
			}
      //  }
		
	
		/*stage('Publish Docker Image') {*/
            container('docker') {
              withCredentials([[$class: 'UsernamePasswordMultiBinding',
                credentialsId: 'gitlab',
                usernameVariable: 'DOCKER_HUB_USER',
                passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
                    sh ('docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASSWORD} '+GCR_HUB_ACCOUNT)
                    sh '''
				committerEmail='''+ committerEmail +'''
				BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
				docker push localhost:32121/root/docker_registry/java:${BUILD_NUMBER}${BUILDUSER}${deploy_env}
				'''
                }
            }
			/*}*/
		//stage('Delete Jenkinslave Service') {
            container('kubectl') {
            sh """
                kubectl delete svc '${label}'
            """
            }
        //}
		/*stage ('Kubectl Check')
           {*/
               container('kubectl')
               {
         
         
           sh 'kubectl get nodes  >> nodes.txt'
            sh label: '', script: '''
            if grep -q master nodes.txt; then
            echo "master found" >> master.txt
            else
            echo not found
            fi
           echo cat master.txt
           '''
               }
          
          if (fileExists('master.txt'))
          {           
            echo " It is  non managed kubernetes service hence executing kube audit and kube bench stages"
            /*stage('Kube-Bench Scan') {*/
            container('kubectl') {       
            sh '''
			v=`kubectl version --short | grep -w 'Server Version'`
			v1=`echo $v | awk -F: '{print $2}'`
			v2=`echo $v1 | sed 's/v//g'`
			versionserver=`echo $v2 | cut -f1,2 -d'.'`
			echo kubernetes version is ${versionserver}
            kubectl run --rm -i kube-bench-master-frontend-${BUILD_ID} --image=aquasec/kube-bench:latest --restart=Never --overrides="{ \\"apiVersion\\": \\"v1\\", \\"spec\\": { \\"hostPID\\": true, \\"nodeSelector\\": { \\"kubernetes.io/role\\": \\"master\\" }, \\"tolerations\\": [ { \\"key\\": \\"node-role.kubernetes.io/master\\", \\"operator\\": \\"Exists\\", \\"effect\\": \\"NoSchedule\\" } ] } }" -- master --version ${versionserver}
            kubectl run --rm -i kube-bench-node-frontend-${BUILD_ID} --image=aquasec/kube-bench:latest --restart=Never --overrides="{ \\"apiVersion\\": \\"v1\\", \\"spec\\": { \\"hostPID\\": true } }" -- node --version ${versionserver}
            '''
            }      
            /*}*/
           /* stage('Kubeaudit Scan') {*/
                  container('kubeaudit') {       
                  sh 'kubeaudit -a allowpe'
                }
            /*}*/
        }
        else
        {
            echo " It is Managed Kubernetes service hence executing kubebench and kube-audit only on nodes "
            /*stage('Kube-Bench Scan') {*/
            container('kubectl') {   
           
            sh '''
			v=`kubectl version --short | grep -w 'Server Version'`
			v1=`echo $v | awk -F: '{print $2}'`
			v2=`echo $v1 | sed 's/v//g'`
			versionserver=`echo $v2 | cut -f1,2 -d'.'`
			echo kubernetes version is ${versionserver}
            kubectl run --rm -i kube-bench-node-frontend-${BUILD_ID} --image=aquasec/kube-bench:latest --restart=Never --overrides="{ \\"apiVersion\\": \\"v1\\", \\"spec\\": { \\"hostPID\\": true } }" -- node --version ${versionserver}
            '''
            }
            /*}*/
             /* stage('Kubeaudit Scan') {*/
                  container('kubeaudit') {       
                  sh 'kubeaudit -a allowpe'
                }
				
          /* }*/
           }
           /*}*/
	if ( "${deploy_env}" == "dev")
	{		
		stage('Application_Deploy') {
			container('docker')
			{
				sh '''
                committerEmail='''+ committerEmail +'''
                BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
                docker build -t localhost:32121/root/docker_registry/java:${BUILD_NUMBER}${BUILDUSER}${deploy_env} .
                '''
				withCredentials([[$class: 'UsernamePasswordMultiBinding',
                credentialsId: 'gitlab',
                usernameVariable: 'DOCKER_HUB_USER',
                passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
                    sh ('docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASSWORD} '+GCR_HUB_ACCOUNT )
				sh '''
				committerEmail='''+ committerEmail +'''
				BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
				docker push localhost:32121/root/docker_registry/java:${BUILD_NUMBER}${BUILDUSER}${deploy_env}
				'''
			    }
            }
			/*container('kubectl') {
			   sh("kubectl apply -f namespace.yaml")
			   sh 'kubectl get secret -n pet-clinic >> nodcred.txt'
			   sh label: '', script: '''
				if grep -q nodecred nodcred.txt; then
				echo "nodecred found " >> pullsecret.txt
				else
			    echo "nodecred not found "		
				fi
				'''
                if (fileExists('pullsecret.txt'))
		  		{
			  	 echo "nodecred already exists"
		  		}
		  		else
		  		{
				echo "nodecred does not exists"
				withCredentials([[$class: 'UsernamePasswordMultiBinding',
					credentialsId: 'gitlab',
					usernameVariable: 'DOCKER_HUB_USER',
                passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
									
					sh("kubectl create secret docker-registry nodecred --docker-server=localhost:32121 --docker-username=${DOCKER_HUB_USER} --docker-password=${DOCKER_HUB_PASSWORD} -n pet-clinic")
				  }
		 		}
				
			}*/
        }
              /*container('kubectl') {
            try{
              sh("kubectl get deployment/${K8S_DEPLOYMENT_NAME} -n pet-clinic")
              if(true){
				
				sh("kubectl delete deployment/${K8S_DEPLOYMENT_NAME} -n pet-clinic")
				sh("sed -i 's/imagetag/${IMAGE_TAG}/g' pet-clinic.yaml")
				sh("kubectl apply -f pet-clinic.yaml -n pet-clinic")
				sleep 30
              }
              } 
              catch(e){
			  sh("sed -i 's/imagetag/${IMAGE_TAG}/g' pet-clinic.yaml")
              sh("kubectl apply -f pet-clinic.yaml -n pet-clinic")
			  sleep 30
			  
              echo "deploying"
              }
			  //sh("kubectl apply -f pet-clinic.yaml -n pet-clinic")
              sh ("kubectl get pods -n pet-clinic")
              sh( "kubectl get svc ${K8S_DEPLOYMENT_NAME} -n pet-clinic")  
             sh ("kubectl rollout status deployment/${K8S_DEPLOYMENT_NAME} -n pet-clinic")
             sleep 60 // seconds 
             LB = sh (returnStdout: true, script: '''kubectl get svc pet-clinic -n pet-clinic -o jsonpath="{.status.loadBalancer.ingress[*]['ip', 'hostname']}" ''')
             echo "LB: ${LB}"
             def loadbalancer = "http://"+LB
             echo "loadbalancer: ${loadbalancer}"
			 echo "application_url: ${loadbalancer}"
			 //the below paths in sed command are application specific and will change according to the application used
			 sh("sed -i 's|appUrl|${loadbalancer}|g' ${WORKSPACE}/Gatling/src/test/scala/PetClinic.scala")
			 sh("sed -i 's|applicationURL|${loadbalancer}|g' ${WORKSPACE}/petclinic.xml")
			 sh("sed -i 's|applicationURL|${loadbalancer}|g' ${WORKSPACE}/src/test/java/petclinic/PageObjMethods/PetclinicPage.java")
			 sh("sed -i 's|applicationURL|${LB}|g' ${WORKSPACE}/JMeter/petclinic.jmx")
             sleep 15			 
				}*/
			container('kubectl'){
                        sh ("sed -i 's/tomcat-deployenv/tomcat${deploy_env}/g' tomcat.yaml")
				        sh ("sed -i 's/deployenv/${deploy_env}/g' tomcat.yaml")
				        sh ("sed -i 's/tomcat-deployenv/tomcat${deploy_env}/g' tomcat-svc.yaml")
                        sh("kubectl apply -f namespace.yaml")
                    
                        wrap([$class: 'BuildUser']) 
                        {
                            sh '''
                            committerEmail='''+ committerEmail +'''
                            BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
                            kubectl get secret -n ${BUILDUSER} >> nodcred.txt
                            '''
                            sh '''
                            if grep -q gcrcred nodcred.txt; then
                            echo "gcrcred found " >> pullsecret.txt
                            else
                            echo "gcrcred not found "		
                            fi
                            '''
                            if (fileExists('pullsecret.txt'))
                            {
                                echo "gcrcred already exists"
                            }
                            else
                            {
                                echo "gcrcred does not exists"
                                withCredentials([[$class: 'UsernamePasswordMultiBinding',
                                  credentialsId: 'gitlab',
                                   usernameVariable: 'DOCKER_HUB_USER',
                                    passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
                                        
                                        sh '''
                                        committerEmail='''+ committerEmail +'''
                                        BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
                                        kubectl create secret docker-registry gcrcred --docker-server=localhost:32121 --docker-username=${DOCKER_HUB_USER} --docker-password=${DOCKER_HUB_PASSWORD} -n ${BUILDUSER}
                                        '''
                                }
                            }
                            try{
                            
                                sh '''
                                committerEmail='''+ committerEmail +'''
                                BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
                                kubectl get deployment/tomcat${deploy_env} -n ${BUILDUSER}
                                '''
                                
                                if(true)
                                {
                                    sh '''
                                    committerEmail='''+ committerEmail +'''
                                    BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
                                    kubectl delete deployment tomcatdev -n ${BUILDUSER}
				                    kubectl apply -f tomcat.yaml
                                    kubectl apply -f tomcat-svc.yaml
                            	    '''
                                }
                            }
                            catch(e)
                            {
                                sh("kubectl apply -f tomcat.yaml")
				                sh("kubectl apply -f tomcat-svc.yaml")
                                sh 'sleep 60'
                                sh '''
                                committerEmail='''+ committerEmail +'''
                                BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
                                kubectl get svc tomcat${deploy_env} -n ${BUILDUSER}                                
                                '''
                            }
                                LB = sh (returnStdout: true, script: '''committerEmail='''+ committerEmail +''' ; BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`;kubectl get svc tomcat${deploy_env} -n ${BUILDUSER} -o jsonpath="{.status.loadBalancer.ingress[*]['ip', 'hostname']}" ''')
                                echo "LB: ${LB}"
                                def loadbalancer = "http://"+LB
                                echo "application_url: ${loadbalancer}"
                            //the below paths in sed command are application specific and will change according to the application used
                            sh("sed -i 's|appUrl|${loadbalancer}|g' ${WORKSPACE}/Gatling/src/test/scala/PetClinic.scala")
                            sh("sed -i 's|applicationURL|${loadbalancer}|g' ${WORKSPACE}/petclinic.xml")
                            sh("sed -i 's|applicationURL|${loadbalancer}|g' ${WORKSPACE}/src/test/java/petclinic/PageObjMethods/PetclinicPage.java")
                            sh("sed -i 's|applicationURL|${LB}|g' ${WORKSPACE}/JMeter/petclinic.jmx")
                            	
                            sleep 40
                        }
                    }
			}
            else
            {
                stage('deploy to dev')
                {
                    echo "dev is not selected"
                }
            }
            stage("Functional_Testing")
     {
         	 try
		{
          // git changelog: false, poll: false, url: 'https://github.com/akilagithub/seleniumreport.git'
	
        }					
          catch(Exception e)
        {
        	  sh 'echo e'
        }
        finally
         {
          publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '', reportFiles: 'LatestTestReportchromereport.html', reportName: 'Selenium Report', reportTitles: 'LatestTestReportchromereport.html']) 
           } 
    }
			
		 stage("Regression_Testing")
  {
    
        
		 try
		{
         // git changelog: false, poll: false, url: 'https://github.com/akilagithub/seleniumreport.git'
	
        }					
          catch(Exception e)
        {
        	  sh 'echo e'
        }
        finally
         {
          cucumber buildStatus: "UNSTABLE", 
     fileIncludePattern: "**/cucumber.json"
     jsonReportDirectory: 'targetselenium'
           } 
	}
   	
	
	stage("Performance Testing")
  {
    container('jmeter') 
	{
               
                //Performance test with JMeter
                    sh '''
                    cp -r ./lib/ /jmeter/
                    echo server.rmi.create=false >> /jmeter/bin/jmeter.properties
                    echo server.rmi.ssl.disable=true >> /jmeter/bin/jmeter.properties
                    '''
                    sh '/jmeter/bin/jmeter.sh -n -t ./JMeter/petclinic.jmx -l PerformanceResult.jtl'
                    perfReport errorFailedThreshold: 0, errorUnstableThreshold: 0, compareBuildPrevious: true, configType: 'PRT', graphType: 'MRT', modePerformancePerTestCase: true, modeThroughput: true, percentiles: '0,50,90,100', sourceDataFiles: 'PerformanceResult.jtl'
                    sh 'cat jmeter.log'
                 
     }
}
	
	




//stage("Gatling")
  //{
        
        /*
	    container('maven')
		{
        	 sh '''
        	 cd $(pwd)/Gatling
             mvn gatling:test -Dgatling.simulationClass=PetClinic
                '''  
             gatlingArchive()
        	 sh "ls -ll"
           
          
       }						
        */
  // }

            if ( "${deploy_env}" == "staging")
	{		
		stage('Deploy to staging') {
			container('docker')
			{
				sh '''
                committerEmail='''+ committerEmail +'''
                BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
                docker build -t localhost:32121/root/docker_registry/java:${BUILD_NUMBER}${BUILDUSER}${deploy_env} .
                '''
				withCredentials([[$class: 'UsernamePasswordMultiBinding',
                credentialsId: 'gitlab',
                usernameVariable: 'DOCKER_HUB_USER',
                passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
                    sh ('docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASSWORD} '+GCR_HUB_ACCOUNT )
				sh '''
				committerEmail='''+ committerEmail +'''
				BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
				docker push localhost:32121/root/docker_registry/java:${BUILD_NUMBER}${BUILDUSER}${deploy_env}
				'''
			    }
            }
			container('kubectl') {
			   sh("kubectl apply -f namespace.yaml")
			   sh 'kubectl get secret -n pet-clinic >> nodcred.txt'
			   sh label: '', script: '''
				if grep -q nodecred nodcred.txt; then
				echo "nodecred found " >> pullsecret.txt
				else
			    echo "nodecred not found "		
				fi
				'''
                if (fileExists('pullsecret.txt'))
		  		{
			  	 echo "nodecred already exists"
		  		}
		  		else
		  		{
				echo "nodecred does not exists"
				withCredentials([[$class: 'UsernamePasswordMultiBinding',
					credentialsId: 'gitlab',
					usernameVariable: 'DOCKER_HUB_USER',
                passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
									
					sh("kubectl create secret docker-registry nodecred --docker-server=localhost:32121 --docker-username=${DOCKER_HUB_USER} --docker-password=${DOCKER_HUB_PASSWORD} -n pet-clinic")
				  }
		 		}
				
			}
        }
            
			container('kubectl'){
                        sh ("sed -i 's/tomcat-deployenv/tomcat${deploy_env}/g' tomcat.yaml")
				        sh ("sed -i 's/deployenv/${deploy_env}/g' tomcat.yaml")
				        sh ("sed -i 's/tomcat-deployenv/tomcat${deploy_env}/g' tomcat-svc.yaml")
                        sh("kubectl apply -f namespace.yaml")
                    
                        wrap([$class: 'BuildUser']) 
                        {
                            sh '''
                            committerEmail='''+ committerEmail +'''
                            BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
                            kubectl get secret -n ${BUILDUSER} >> nodcred.txt
                            '''
                            sh '''
                            if grep -q gcrcred nodcred.txt; then
                            echo "gcrcred found " >> pullsecret.txt
                            else
                            echo "gcrcred not found "		
                            fi
                            '''
                            if (fileExists('pullsecret.txt'))
                            {
                                echo "gcrcred already exists"
                            }
                            else
                            {
                                echo "gcrcred does not exists"
                                withCredentials([[$class: 'UsernamePasswordMultiBinding',
                                  credentialsId: 'gitlab',
                                   usernameVariable: 'DOCKER_HUB_USER',
                                    passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
                                        
                                        sh '''
                                        committerEmail='''+ committerEmail +'''
                                        BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
                                        kubectl create secret docker-registry gcrcred --docker-server=localhost:32121 --docker-username=${DOCKER_HUB_USER} --docker-password=${DOCKER_HUB_PASSWORD} -n ${BUILDUSER}
                                        '''
                                }
                            }
                            try{
                            
                                sh '''
                                committerEmail='''+ committerEmail +'''
                                BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
                                kubectl get deployment/tomcat${deploy_env} -n ${BUILDUSER}
                                '''
                                
                                if(true)
                                {
                                    sh '''
                                    committerEmail='''+ committerEmail +'''
                                    BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
                                    kubectl delete deployment tomcatstaging -n ${BUILDUSER}
				                    kubectl apply -f tomcat.yaml
                                    kubectl apply -f tomcat-svc.yaml
                                    '''
                                }
                            }
                            catch(e)
                            {
                                sh("kubectl apply -f tomcat.yaml")
				                sh("kubectl apply -f tomcat-svc.yaml")
                                sh 'sleep 60'
                                sh '''
                                committerEmail='''+ committerEmail +'''
                                BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
                                kubectl get svc tomcat${deploy_env} -n ${BUILDUSER}
                                '''
                            }
                            LB = sh (returnStdout: true, script: '''committerEmail='''+ committerEmail +''' ; BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`;kubectl get svc tomcat${deploy_env} -n ${BUILDUSER} -o jsonpath="{.status.loadBalancer.ingress[*]['ip', 'hostname']}" ''')
                                echo "LB: ${LB}"
                                def loadbalancer = "http://"+LB
                                echo "application_url: ${loadbalancer}"
                            //the below paths in sed command are application specific and will change according to the application used
                            sh("sed -i 's|appUrl|${loadbalancer}|g' ${WORKSPACE}/Gatling/src/test/scala/PetClinic.scala")
                            sh("sed -i 's|applicationURL|${loadbalancer}|g' ${WORKSPACE}/petclinic.xml")
                            sh("sed -i 's|applicationURL|${loadbalancer}|g' ${WORKSPACE}/src/test/java/petclinic/PageObjMethods/PetclinicPage.java")
                            sh("sed -i 's|applicationURL|${LB}|g' ${WORKSPACE}/JMeter/petclinic.jmx")
                            	
                            sleep 40
                        }
                    }
			}
            else
            {
                stage('deploy to staging')
                {
                    echo "staging is not selected"
                }
            }
            if ( "${deploy_env}" == "prod")
	{		
		stage('Deploy to prod') {
			container('docker')
			{
				sh '''
                committerEmail='''+ committerEmail +'''
                BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
                docker build -t localhost:32121/root/docker_registry/java:${BUILD_NUMBER}${BUILDUSER}${deploy_env} .
                '''
				withCredentials([[$class: 'UsernamePasswordMultiBinding',
                credentialsId: 'gitlab',
                usernameVariable: 'DOCKER_HUB_USER',
                passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
                    sh ('docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASSWORD} '+GCR_HUB_ACCOUNT )
				sh '''
				committerEmail='''+ committerEmail +'''
				BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
				docker push localhost:32121/root/docker_registry/java:${BUILD_NUMBER}${BUILDUSER}${deploy_env}
				'''
			    }
            }
			container('kubectl') {
			   sh("kubectl apply -f namespace.yaml")
			   sh 'kubectl get secret -n pet-clinic >> nodcred.txt'
			   sh label: '', script: '''
				if grep -q nodecred nodcred.txt; then
				echo "nodecred found " >> pullsecret.txt
				else
			    echo "nodecred not found "		
				fi
				'''
                if (fileExists('pullsecret.txt'))
		  		{
			  	 echo "nodecred already exists"
		  		}
		  		else
		  		{
				echo "nodecred does not exists"
				withCredentials([[$class: 'UsernamePasswordMultiBinding',
					credentialsId: 'gitlab',
					usernameVariable: 'DOCKER_HUB_USER',
                passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
									
					sh("kubectl create secret docker-registry nodecred --docker-server=localhost:32121 --docker-username=${DOCKER_HUB_USER} --docker-password=${DOCKER_HUB_PASSWORD} -n pet-clinic")
				  }
		 		}
				
			}
        }
            
			container('kubectl'){
                        sh ("sed -i 's/tomcat-deployenv/tomcat${deploy_env}/g' tomcat.yaml")
				        sh ("sed -i 's/deployenv/${deploy_env}/g' tomcat.yaml")
				        sh ("sed -i 's/tomcat-deployenv/tomcat${deploy_env}/g' tomcat-svc.yaml")
                        sh("kubectl apply -f namespace.yaml")
                    
                        wrap([$class: 'BuildUser']) 
                        {
                            sh '''
                            committerEmail='''+ committerEmail +'''
                            BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
                            kubectl get secret -n ${BUILDUSER} >> nodcred.txt
                            '''
                            sh '''
                            if grep -q gcrcred nodcred.txt; then
                            echo "gcrcred found " >> pullsecret.txt
                            else
                            echo "gcrcred not found "		
                            fi
                            '''
                            if (fileExists('pullsecret.txt'))
                            {
                                echo "gcrcred already exists"
                            }
                            else
                            {
                                echo "gcrcred does not exists"
                                withCredentials([[$class: 'UsernamePasswordMultiBinding',
                                  credentialsId: 'gitlab',
                                   usernameVariable: 'DOCKER_HUB_USER',
                                    passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
                                        
                                        sh '''
                                        committerEmail='''+ committerEmail +'''
                                        BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
                                        kubectl create secret docker-registry gcrcred --docker-server=localhost:32121 --docker-username=${DOCKER_HUB_USER} --docker-password=${DOCKER_HUB_PASSWORD} -n ${BUILDUSER}
                                        '''
                                }
                            }
                            try{
                            
                                sh '''
                                committerEmail='''+ committerEmail +'''
                                BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
                                kubectl get deployment/tomcat${deploy_env} -n ${BUILDUSER}
                                '''
                                
                                if(true)
                                {
                                    sh '''
                                    committerEmail='''+ committerEmail +'''
                                    BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
                                    kubectl delete deployment tomcatprod -n ${BUILDUSER}
				                    kubectl apply -f tomcat.yaml
                                    kubectl apply -f tomcat-svc.yaml
                                    '''
                                }
                            }
                            catch(e)
                            {
                                sh("kubectl apply -f tomcat.yaml")
				                sh("kubectl apply -f tomcat-svc.yaml")
                                sh 'sleep 60'
                                sh '''
                                committerEmail='''+ committerEmail +'''
                                BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`
                                kubectl get svc tomcat${deploy_env} -n ${BUILDUSER}
                                '''
                            }
                            LB = sh (returnStdout: true, script: '''committerEmail='''+ committerEmail +''' ; BUILDUSER=`echo $committerEmail | awk -F@ '{print $1}'`;kubectl get svc tomcat${deploy_env} -n ${BUILDUSER} -o jsonpath="{.status.loadBalancer.ingress[*]['ip', 'hostname']}" ''')
                                echo "LB: ${LB}"
                                def loadbalancer = "http://"+LB
                                echo "application_url: ${loadbalancer}"
                            //the below paths in sed command are application specific and will change according to the application used
                            sh("sed -i 's|appUrl|${loadbalancer}|g' ${WORKSPACE}/Gatling/src/test/scala/PetClinic.scala")
                            sh("sed -i 's|applicationURL|${loadbalancer}|g' ${WORKSPACE}/petclinic.xml")
                            sh("sed -i 's|applicationURL|${loadbalancer}|g' ${WORKSPACE}/src/test/java/petclinic/PageObjMethods/PetclinicPage.java")
                            sh("sed -i 's|applicationURL|${LB}|g' ${WORKSPACE}/JMeter/petclinic.jmx")
                            
                            sleep 40
                        }
                    }
			}
            else
            {
                stage('deploy to prod')
                {
                    echo "prod is not selected"
                }
            } 



			
		/*	stage('OWASP Webscan') {
                    container('docker') {
                       try{
					sh ("docker run -t owasp/zap2docker-stable zap-baseline.py -t 'http://${LB}' ")
                   }catch(Exception e){
                       sh 'echo "error"'
                   }
                    sh 'echo "Check"'
                   }
                   
               }
          stage ('Nexus Uploader'){
		    container('maven'){
		    echo "Uploading selenium reports to Nexus"
            sh "echo ${JOBNAME}"
		    withCredentials([usernamePassword(credentialsId: 'NEXUS', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
            sh( "curl --upload-file ${WORKSPACE}/test-output/LatestTestReportchromereport.html -u $USERNAME:$PASSWORD -v http://nexus.ethan.svc.cluster.local:8083/nexus/repository/maven-releases/selenium-script/${JOBNAME}/$BUILD_ID/")
            }
		}
	} */
			
	
	
        }
	

	currentBuild.result = 'SUCCESS'
	echo "RESULT: ${currentBuild.result}"
	echo "Finished: ${currentBuild.result}"
              
	} catch (Exception err) {
        currentBuild.result = 'FAILURE'
        
        echo "RESULT: ${currentBuild.result}"
        echo "Finished: ${currentBuild.result}"
        
		}               
     logstashSend failBuild: false, maxLines: -1
     }
     }