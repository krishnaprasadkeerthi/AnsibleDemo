pipeline {
   
   tools {
        maven 'Maven3'
    }
   parameters { 
              string(name: 'ArtifactVersion', defaultValue: '0.1.1', description: "Please Provide Artifact Version")
              string(name: 'nexus_ip', description: "Please Provide nexus ip")
   }                  
   environment {     
            NEXUS_VERSION = "nexus3"
            NEXUS_PROTOCOL = "http"
            NEXUS_URL = "172.31.85.199:8081"
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
                sh 'mvn clean deploy'           
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
             script {
                      sh """ sudo ansible-playbook -i myhosts ansible.yml --vault-password-file /etc/ansible/config.txt --extra-vars '{"ArtifactVersion": "${params.ArtifactVersion}", "nexus-ip": "${params.nexus_ip}"}' """
             }   
          } 
       }
    }      
 }


