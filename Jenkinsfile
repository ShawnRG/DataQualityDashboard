pipeline {
    agent {
        label 'amzl-honeur-upgrade'
    }

    parameters {
        string defaultValue: 'https://r-package-manager.honeur.org/',description: 'The host url of RStudio Package Manager', name: 'RSTUDIO_PACKAGE_MANAGER_HOST', trim: true
    }

    stages {
        stage('Tag latest changes') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-credentials-shawn', passwordVariable: 'GITHUB_TOKEN', usernameVariable: 'GITHUB_USERNAME')]) {
                    script {
                        descriptionFileString = readFile encoding: 'UTF-8', file: 'DESCRIPTION'
                        oldVersion = (descriptionFileString =~ /Version: (.*)/)[0][1]
                        newVersion = getNewVersion(oldVersion)
                        env.NEW_VERSION = newVersion
                        echo "New version -> ${newVersion}"
                        newDescriptionFileString = descriptionFileString.replaceAll(/Version: .*/, "Version: " + newVersion)
                        echo newDescriptionFileString
                        writeFile encoding: 'UTF-8', file: 'DESCRIPTION', text: newDescriptionFileString
                        sh "git add DESCRIPTION"
                        sh "git commit -m \"Created release ${newVersion}\""
                        sh "git tag -a v${newVersion} -m 'Version ${newVersion}'"
                        sh "git push https://${env.GITHUB_USERNAME}:${env.GITHUB_TOKEN}@github.com/ShawnRG/DataQualityDashboard"
                        sh "git push https://${env.GITHUB_USERNAME}:${env.GITHUB_TOKEN}@github.com/ShawnRG/DataQualityDashboard --tags"
                    }
                }
            }
        }
        stage('Wait for completed build') {
            steps {
                script {
                    retry(3) {
                        waitUntil {
                            server_version = sh(
                                script: "curl -X GET ${env.RSTUDIO_PACKAGE_MANAGER_HOST}__api__/repos/4/packages/DataQualityDashboard | python -c \"import sys,json; print json.load(sys.stdin)['version']\"",
                                returnStdout: true
                            ).trim()
                            echo "Comparing server version (${server_version}) and new version (${env.NEW_VERSION})"
                            return server_version == env.NEW_VERSION
                        }
                    }
                }
            }
        }

        stage('Run DQD build job') {
            steps {
                build job: 'DataQualityDashboard-Image', parameters: [string(name: 'VERSION', value: params.VERSION)]
            }
        }
    }

}

def getNewVersion(oldVersion) {
    versionParts = oldVersion.split('\\.')
    majorVersionNumber = versionParts[0]
    minorVersionNumber = versionParts[1] as Integer
    return "${majorVersionNumber}.${minorVersionNumber + 1}"
}