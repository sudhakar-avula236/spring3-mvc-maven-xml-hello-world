pipeline {
    agent any
    
    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "maven"
    }
    
    environment {
             // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "ec2-100-26-45-160.compute-1.amazonaws.com:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "spring-app"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "nexus_credentials"
    }
    
    stages {
        stage('getscm') {
            steps {
                git changelog: false, credentialsId: 'github_credentials', poll: false, url: 'https://github.com/sudhakar-avula236/spring3-mvc-maven-xml-hello-world.git'
            }
        }
        stage('build') {
            steps {
                sh "mvn clean package"
            }
            post {
                success {
                    archiveArtifacts 'target/*.war'
                }
            }           
        }
        
        stage("publish to nexus") {
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml";
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path;
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: '${BUILD_NUMBER}',
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
						sh "curl -v ${NEXUS_PROTOCOL}://${NEXUS_URL}/repository/${NEXUS_REPOSITORY}/com/mkyong/${pom.artifactId}/${BUILD_NUMBER}/${pom.artifactId}-${BUILD_NUMBER}.${pom.packaging} -o result.war"
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
		
        stage('deploy') {
            steps {
                withCredentials([usernameColonPassword(credentialsId: 'tomcat_remote_credential', variable: 'tomcat_credentials')]) {
                   sh "curl -v -u ${tomcat_credentials} -T result.war 'http://ec2-18-206-147-65.compute-1.amazonaws.com:8080/manager/text/deploy?path=/springdeploy3&update=true'"   
               }
            }
        }		
    }
}

