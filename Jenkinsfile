/*
 * Requires: https://github.com/RedHatInsights/insights-pipeline-lib
 */

@Library("github.com/RedHatInsights/insights-pipeline-lib") _

// Code coverage failure threshold
codecovThreshold = 0


node {
    cancelPriorBuilds()

    runIfMasterOrPullReq {
        runStages()
    }
}


def runStages() {
    openShift.withNode(image: "docker-registry.default.svc:5000/jenkins/jenkins-slave-base-centos7-python36:latest") {
        // check out source again to get it in this node's workspace
        scmVars = checkout scm

        stage('Pip install') {
            runPipenvInstall(scmVars: scmVars)
        }

        stage('Lint') {
            runPythonLintCheck(pylama: true)
        }

        stage('UnitTest') {
            withStatusContext.unitTest {
                sh "${pipelineVars.userPath}/pipenv run python -m pytest --junitxml=junit.xml --cov=. --cov-report html -s -v"
            }

            junit 'junit.xml'
        }

        stage('Code coverage') {
            checkCoverage(threshold: codecovThreshold)
        }

        if (currentBuild.currentResult == 'SUCCESS') {
            if (env.BRANCH_NAME == 'master') {
                // Stages to run specifically if master branch was updated
            }
        }
    }
}

