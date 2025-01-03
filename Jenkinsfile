pipeline {
    agent any
    tools {
        maven "Maven 3" // Replace with your Maven installation name in Jenkins
        jdk "Java 17"   // Replace with your JDK installation name in Jenkins
    }

    environment {
        // Nexus configurations
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "127.0.0.1:9091"
        NEXUS_REPOSITORY = "WebApp"
        NEXUS_CREDENTIAL_ID = "nexusCredential"
        ARTIFACT_VERSION = "${BUILD_NUMBER}"
    }

    stages {
        stage("Checkout Code") {
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/Adarsh0871/WebApp.git'
                }
            }
        }

        stage("Build Project") {
            steps {
                script {
                    sh "mvn clean package"
                }
            }
        }

        stage("Publish to Nexus") { // Corrected the syntax error here
            steps {
                script {
                    pom = readMavenPom file: "pom.xml"
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    if (filesByGlob.size() > 0) {
                        artifactPath = filesByGlob[0].path
                        artifactExists = fileExists artifactPath

                        if (artifactExists) {
                            echo "Found artifact: ${artifactPath}"
                            echo "Group: ${pom.groupId}, Artifact: ${pom.artifactId}, Version: ${pom.version}"

                            nexusArtifactUploader(
                                nexusVersion: NEXUS_VERSION,
                                protocol: NEXUS_PROTOCOL,
                                nexusUrl: NEXUS_URL,
                                groupId: pom.groupId,
                                version: ARTIFACT_VERSION,
                                repository: NEXUS_REPOSITORY,
                                credentialsId: NEXUS_CREDENTIAL_ID,
                                artifacts: [
                                    [
                                        artifactId: pom.artifactId,
                                        classifier: '',
                                        file: artifactPath,
                                        type: pom.packaging
                                    ]
                                ]
                            )
                        } else {
                            error "Artifact file not found at: ${artifactPath}"
                        }
                    } else {
                        error "No artifacts found in target directory."
                    }
                }
            }
        }

        stage("Execute Ansible Play - CD") {
            agent {
                label 'ansible'
            }
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/Adarsh0871/WebApp.git'
                }
                sh '''
                    ansible-playbook -e vers=${BUILD_NUMBER} roles/site.yml
                '''
            }
        }
    }
}
