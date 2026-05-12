// TeamViewer LiteLLM fork build pipeline.
//
// Mirrors MCO/Publish web-api-swagger-docs (the reference job we copied),
// adapted for the litellm repo where the Dockerfile is at the repo root and
// the image is published to acrplatform2global1.azurecr.io/custom-images/litellm.
//
// The image tag is `<branch>-<shortSha>-<utcStamp>` so it's always unique and
// argocd-environments can pin to an exact build.
pipeline {
    options {
        ansiColor('xterm')
        buildDiscarder logRotator(
            artifactDaysToKeepStr: '60',
            artifactNumToKeepStr: '10',
            daysToKeepStr: '60',
            numToKeepStr: '10'
        )
        disableConcurrentBuilds abortPrevious: true
        skipDefaultCheckout(false)
    }
    agent none
    stages {
        stage('Build Docker') {
            agent {
                node {
                    label "lnxbuildvm"
                }
            }
            steps {
                script {
                    def imageName = "litellm"
                    def shortSha = sh(script: "git rev-parse --short=10 HEAD", returnStdout: true).trim()
                    def stamp = new Date().format("yyyyMMdd-HHmmss", TimeZone.getTimeZone("UTC"))
                    def safeBranch = (env.BRANCH_NAME ?: "unknown").replaceAll("[^A-Za-z0-9_.-]", "-")
                    def tag = "${safeBranch}-${shortSha}-${stamp}"
                    def fullImageName = "acrplatform2global1.azurecr.io/custom-images/${imageName}:${tag}"

                    sh "docker build -t ${fullImageName} ."

                    withCredentials([
                        usernamePassword(
                            credentialsId: 'acrp2usertoken',
                            passwordVariable: 'ACR_PASS',
                            usernameVariable: 'ACR_USER'
                        )
                    ]) {
                        sh label: 'Docker Login', script: 'docker login -u ${ACR_USER} -p ${ACR_PASS} acrplatform2global1.azurecr.io'
                        sh "docker push ${fullImageName}"
                    }

                    echo "Successfully built and pushed: ${fullImageName}"
                    currentBuild.description = fullImageName
                }
            }
        }
    }
}
