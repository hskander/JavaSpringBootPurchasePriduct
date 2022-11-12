pipeline {
    agent any
   environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "192.168.44.200:8081"
        NEXUS_REPOSITORY = "maven-snapshots"
        NEXUS_CREDENTIAL_ID = "hamdisanderNexus"
    }


    stages {

        stage('Clone Git') {
            steps {
                   git url : 'https://github.com/hskander/JavaSpringBootPurchasePriduct.git',
                     credentialsId : 'hamdiskander',
                     branch : 'main'

            }
        }
         stage('clean'){
            steps {
                sh "mvn clean"
            }
        }

         stage('compile'){
            steps {
                sh "mvn compile -e"
            }
        }
        stage('install'){
            steps {
                sh "mvn install -DskipTests"
            }
        }
        stage('MvnPackage'){
            steps {
                sh "mvn package -DskipTests "
            }
        }
         stage('sonar'){
            steps {
                script{ withSonarQubeEnv('skanderSonarQube') {
                     sh "mvn sonar:sonar -DskipTests"
                 }

                }
            }
        }
        stage("Nexus") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
    }
}