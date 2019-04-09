#!/usr/bin/env groovy

// Jenkins managed file ID for nexus settings
def nexusSettingFileId = "cicd-setting-nexus3"

// Jenkins managed credentials id required for uploading binaries to Nexus
def nexusCredentialID = "nexus-credentials"

// Command line parameters for maven build
def buildCmdParams = "clean install -Dmaven.test.skip=true -P"

// Nexus Repository URL
def nexusRepository = 'nexus.devops-srid.svc.cluster.local:8081'

def jar = "batch/target/batch-0.0.1-SNAPSHOT.jar"
def isFail = true
def status = true

// Github URL 
def git_url = "https://github.sec.samsung.net/"

// Github repo of the source code
def git_repo = "VD-SmartTVServer/uca"

// Ansible Job Template Name which deploys the application
def ansibleJobTemplate = "uca_batch_deploy"

// Ansible Tower Server config name as configured in Jenkins
def ansibleTowerServer = "Prod Tower"

// Ansible inventory for eg.. uca_stg, uca_dev
def inventory_file = ''

// maven build profiles for various stages
def stage_map = [dev:"dev_batch",stg:"stg_batch", prd:"prd_batch"]

def ver_no = "0.0.1"

node('master')
{
    stage('Info') {        
        inventory_file = "uca_"+"${params.STAGE}"
        echo inventory_file        
    }
}
node('maven') {
    try {
        stage ('build'){
            
            // Checkout Source Code
            git url: "${git_url}${git_repo}", branch: "$params.BRANCH", credentialsId: "sds-github"
            
            // Fetch Maven Settings and run maven build 
            configFileProvider([configFile(fileId: "${nexusSettingFileId}", variable: 'MAVEN_SETTINGS')]) {
                def stage = stage_map[params.STAGE]
                def buildCmd = "mvn -s ${MAVEN_SETTINGS} ${buildCmdParams}${stage}"
                sh buildCmd
            }
            
            // Stash the binary so that it can be uploaded to stable nexus repository once the release is successfu
            stash includes: "$jar", name: "jars"
            
            // Upload binary to nexus using the Jenkins managed Nexus Credentials
            nexusArtifactUploader(
                nexusVersion: 'nexus3',
                protocol: 'http',
                nexusUrl: "${nexusRepository}",
                groupId: "uca/${params.STAGE}",
                version: '0.0.1',
                repository: 'samsungtv',
                credentialsId: "${nexusCredentialID}",
                artifacts: [
                  [
                    artifactId: 'batch',
                    classifier: 'SNAPSHOT',
                    file: "${jar}",
                    type: 'jar'
                  ]
                ]
            )
        }
        
        // SonarQube Analysis
      /*  stage('Sonarqube analysis') {
            def scannerHome = tool 'sonar-runner-auto';
            print scannerHome
            withSonarQubeEnv('sonarserver') {
                sh "${scannerHome}/bin/sonar-scanner -X\
                -Dsonar.analysis.mode=publish"
            }
        }*/
    } catch(e) {
        echo "Exception raised ${e}"
        status = false
    }
}

/**
* If the build and Sonar Analysis is successful 
* invoke Ansible Job template to deploy the application with extravars,
* file_version: version of the binary
* repo_path: Path in nexus repo from where the binary can be fetched
**/

node('master') {
    try {              
        if(status == true) {    
            stage ('Ansible-Tower UCA deployment') {
                ansibleTower(
                    towerServer: "${ansibleTowerServer}",
                    jobTemplate: "${ansibleJobTemplate}",
                    importTowerLogs: true,
                    inventory: "${inventory_file}",
                    jobTags: '',
                    limit: '',
                    removeColor: false,
                    verbose: true,
                    extraVars: '''---
                    file_version: "0.0.1"
                    repo_path: ""'''
                )
            }
        }
    } catch(e) {
        echo "Exception raised ${e}"
        status = false
    }
}

/**
* In case of successful deployment this step will upload the current binary to stable folder in nexus
* In case of unsuccessful deployment this step will invoke the Ansible Tower Job template to redeploy the 
* application using previous stable build
**/

passedBuilds = []

def lastSuccessfullBuild(build) {
    if(build != null) {
        if(build.result != 'FAILURE') {
            passedBuilds.add(build);
        }
        else
        {
            lastSuccessfullBuild(build.getPreviousBuild());
        }
    }
}

node('master') {
    try {
        if(status == true) {

            // Successful deployment
            stage ('upload stable build to nexus') {
                unstash 'jars'                
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${nexusRepository}",
                    groupId: "uca/${params.STAGE}",
                    version: "${env.BUILD_NUMBER}",
                    repository: 'samsungtv',
                    credentialsId: "${nexusCredentialID}",
                    artifacts: [
                      [
                        artifactId: 'batch',
                        classifier: 'SNAPSHOT',
                        file: "${jar}",
                        type: 'jar'
                      ]
                    ]
                )                
            }
        }else{
            lastSuccessfullBuild(currentBuild.getPreviousBuild());
            if(passedBuilds[0] != null)
            {
                ver_no = passedBuilds[0].number
            }
            def extra_var = "---\nfile_version: \"" + ver_no + "\"\nrepo_path: \"\""
            stage ('Ansible-Tower UCA deployment rollback') {
            ansibleTower(
                towerServer: "${ansibleTowerServer}",
                jobTemplate: "${ansibleJobTemplate}",
                importTowerLogs: true,
                inventory: "${inventory_file}",
                jobTags: '',
                limit: '',
                removeColor: false,
                verbose: true,
                extraVars: "${extra_var}"                
                )
            }
        }
    } catch(e) {
        echo "Exception raised ${e}"
        status = false
    }
    finally {
        currentBuild.result = 'SUCCESS'
        if(status == false)
        {
            currentBuild.result = 'FAILURE'
        }        
    }    
}
