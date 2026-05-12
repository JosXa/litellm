// Builds the TeamViewer LiteLLM fork and publishes the resulting image to
// acrplatform2global1.azurecr.io/custom-images/litellm. Image tag mirrors the
// upstream litellm version with a `-tv_fork.<BUILD_NUMBER>` suffix so each
// TV build is uniquely addressable from argocd-environments and visibly
// distinct from upstream Berri tags.
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
                    def litellmVersion = sh(
                        script: "grep -m1 '^version' pyproject.toml | sed -E 's/^version[[:space:]]*=[[:space:]]*\"([^\"]+)\".*/\\1/'",
                        returnStdout: true
                    ).trim()
                    if (!(litellmVersion ==~ /\d+\.\d+\.\d+/)) {
                        error("Could not parse litellm version from pyproject.toml (got: '${litellmVersion}')")
                    }

                    def safeBranch = (env.BRANCH_NAME ?: "unknown").replaceAll("[^A-Za-z0-9.]", "-")
                    def base = "v${litellmVersion}-tv_fork.${env.BUILD_NUMBER}"
                    def tag = (safeBranch == "main") ? base : "${base}-${safeBranch}"
                    def fullImageName = "acrplatform2global1.azurecr.io/custom-images/litellm:${tag}"

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
