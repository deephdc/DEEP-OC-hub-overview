#!/usr/bin/groovy

@Library(['github.com/indigo-dc/jenkins-pipeline-library@1.2.3']) _

pipeline {
    agent {
        label 'docker-build'
    }

    stages {
        stage('Docker image delete tag') {
            when {
                anyOf {
                   branch 'master'
               }
            }
            environment {
                DOCKER_CREDS = credentials('indigobot')
            }
            steps{
                checkout scm
                script {
                    // same REPO for DockerHub and GitHub
                    def REPO = "deep-oc-semseg_vaihingen"
                    def REPO_SUMMARY = "Dogs Breed detector based on deep learning, uses DEEPaaS API."
                    
                    // === everything below should not be changed ===
                    // same ORG for DockerHub and GitHub                      
                    def ORG = "deephdc"                    
                    def URL_HUB = "https://hub.docker.com/v2"
                    def DOCKER_REPO_URL="${URL_HUB}/repositories/${ORG}/${REPO}/"
                    def README_URL = "https://raw.githubusercontent.com/${ORG}/${REPO}/master/README.md"

                    echo "${README_URL}"

                    // get Docker Hub Token
                    //  .first install jq
                    sh "wget -O jq https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64 && chmod +x ./jq"
                    //  .obtain TOKEN. For some reason have to trancate end-of-line
                    TOKEN = sh(
                        script: "curl -s -H \"Content-Type: application/json\" -X POST -d '{\"username\": \"${DOCKER_CREDS_USR}\", \"password\": \"${DOCKER_CREDS_PSW}\"}' ${URL_HUB}/users/login/ | ./jq -r .token | tr -d '\n\t'",
                        returnStdout: true,
                    )               

                    def WORKSPACE = pwd()
                    def README_PATH = "${WORKSPACE}/_README.md"
                    // download GitHub README.md
                    sh("curl -o ${README_PATH} ${README_URL}")
                    // update Overview at Docker Hub with the README.md
                    RESPONSE_CODE = sh(script:
                        "curl -s --write-out %{response_code} --output /dev/null -H \"Authorization: JWT ${TOKEN}\" -X PATCH --data-urlencode full_description@${README_PATH} ${DOCKER_REPO_URL}",
                        returnStdout: true,
                    )

                    echo "[INFO] Received response code: ${RESPONSE_CODE}";

                    // update short description, aka summary
                    sh("curl -s --write-out %{response_code} --output /dev/null -H \"Authorization: JWT ${TOKEN}\" -X PATCH --data-urlencode \"description=${REPO_SUMMARY}\" ${DOCKER_REPO_URL}")
                }
            }
         }
    }

    post {
         always {
            // cleanup
            deleteDir()
         }
    }

}
