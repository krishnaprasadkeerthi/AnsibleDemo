pipeline {
   
   tools {
        maven 'Maven3'
    }
   environment {     
            NEXUS_VERSION = "nexus3"
            NEXUS_PROTOCOL = "http"
            NEXUS_URL = "172.31.86.198:8081"
            NEXUS_REPOSITORY = "maven-releases"
            NEXUS_CREDENTIAL_ID = "nexus"
    } 
    agent any
    stages {
        stage('Cloning Git') {
            steps {
                    checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/AbdulShukur007/Ansible-demo.git']]])
            }
        }  
        stage('Maven Build'){
            steps {
                sh 'mvn clean install'           
            }
        }
        stage('Push to Nexus'){
           steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(nexusVersion: NEXUS_VERSION, protocol: NEXUS_PROTOCOL, nexusUrl: NEXUS_URL, groupId: pom.groupId, version: pom.version, repository: NEXUS_REPOSITORY, credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId, classifier: '', file: artifactPath, type: pom.packaging],
                                [artifactId: pom.artifactId, classifier: '', file: "pom.xml", type: "pom"]
                            ]);} 
                        else {
                               error "*** File: ${artifactPath}, could not be found";
                    }
               }  
           }  
        }
       stage('Ansible Deploy'){
          steps { 
                  ansiblePlaybook credentialsId: 'ansible', disableHostKeyChecking: true, installation: 'ansible', inventory: 'myhosts', playbook: 'ansible.yml' -vvv
                 } 
       }   
    }      
 }
