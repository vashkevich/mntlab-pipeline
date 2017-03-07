node('host'){
    tool name: 'java8', type: 'jdk'
    tool name: 'gradle3.3', type: 'gradle'
  
    prepStatus = "\nPreparation: FAILURE"
    buildStatus = "\nBuilding code: FAILURE"
    testStatus = "\nTesting: FAILURE"
    trigStatus = "\nTriggering job and fetching artefact Stage: FAILURE"
    packStatus = "\nPackaging and Publishing results: FAILURE"
    aprStatus = "\nAsking for manual approval: FAILURE"
    deployStatus = "\nDeployment: FAILURE"
    

withEnv(["JAVA_HOME=${ tool 'java8' }", "PATH+GRADLE=${tool 'gradle3.3'}/bin", "PATH+JAVA=${tool 'java8'}/bin"]) { 

try{ 
stage ('Preparation')
    {
    git branch: 'akutsko', url: 'https://github.com/MNT-Lab/mntlab-pipeline.git'
    prepStatus = "\nPreparation: SACCESS"
    }

stage ('Building code')
    {
    sh 'gradle build'
    buildStatus = "\nBuilding code: SACCESS"
    }
stage ('Testing code'){
    parallel Unit_Test: {sh 'gradle test'
             }, Jacoco_Test: {sh 'gradle jacoco'
             }, cucumber_Test: {sh 'gradle cucumber'
             }
    testStatus = "\nTesting: SACCESS"
    }
stage ('Triggering job and fetching artefact after finishing'){
    build job: 'MNTLAB-akutsko-child1-build-job', parameters: [string(name: 'BRANCH_NAME', value: 'akutsko')], quietPeriod: 0
    step ([$class: 'CopyArtifact',
          projectName: 'MNTLAB-akutsko-child1-build-job',
          filter: 'akutsko_dsl_script.tar.gz']);
    trigStatus = "\nTriggering job and fetching artefact Stage: SACCESS"
    }
stage ('Packaging and Publishing results'){
    sh 'cp build/libs/$(basename "$PWD").jar akutsko-v1.${BUILD_NUMBER}.jar'
    sh 'tar -xf akutsko_dsl_script.tar.gz jobs.groovy'
    sh 'tar -cvzf pipeline-akutsko-${BUILD_NUMBER}.tar.gz jobs.groovy Jenkinsfile  akutsko-v1.${BUILD_NUMBER}.jar'
    archiveArtifacts artifacts: 'pipeline-akutsko-${BUILD_NUMBER}.tar.gz', excludes: null
    packStatus = "\nPackaging and Publishing results: SACCESS"
    }
stage ('Asking for manual approval'){
    input message: 'Do you want to deploy this artefact?', ok: 'Deploy'
    aprStatus = "\nAsking for manual approval: SACCESS"
    } 
stage ('Deployment'){
    sh 'tar -xf pipeline-akutsko-${BUILD_NUMBER}.tar.gz akutsko-v1.${BUILD_NUMBER}.jar'
    sh 'java -jar akutsko-v1.${BUILD_NUMBER}.jar'
    deployStatus = "\nDeployment: SACCESS"
    }
  }
catch (ERROR){}
    stage('Sending status') 
   {
	echo prepStatus + buildStatus + testStatus + trigStatus + packStatus + aprStatus + deployStatus
   }

}
}
