pipeline {
    agent {
        label 'amzl-honeur-upgrade'
    }

    parameters {
        string defaultValue: 'http://34.246.134.235:4242/',description: 'The host url of RStudio Package Manager', name: 'RSTUDIO_PACKAGE_MANAGER_HOST', trim: true
    }

    stages {
//         stage('Check latest commit is tagged') {
//             steps {
//                 script {
//                     try {
//                         sh 'git describe --exact-match'
//                         echo 'Latest commit is tagged, skipping...'
//                         env.SKIP = 'TRUE'
//                     } catch(Exception e) {
//                         env.SKIP = 'FALSE'
//                     }
//                 }
//             }
//         }
        stage('Tag latest changes') {
            steps {
//                 when {
//                     expression {
//                         return env.SKIP == 'FALSE'
//                     }
//                 }
                withCredentials([usernamePassword(credentialsId: 'github-credentials-shawn', passwordVariable: 'GITHUB_TOKEN', usernameVariable: 'GITHUB_USERNAME')]) {
                    script {
                        descriptionFileString = readFile encoding: 'UTF-8', file: 'DESCRIPTION'
                        oldVersion = (descriptionFileString =~ /Version: (.*)/)[0][1]
                        newVersion = getNewVersion(oldVersion)
                        newDescriptionFileString = descriptionFileString.replaceAll(/Version: (.*)/, "Version: " + newVersion)
                        writeFile encoding: 'UTF-8', file: 'DESCRIPTION', text: "${newDescriptionFileString}"
                        sh "git add DESCRIPTION"
                        sh "git tag -a v${newVersion} -m 'Version ${newVersion}'"
                        sh "git push https://${env.GITHUB_USERNAME}:${env.GITHUB_TOKEN}@github.com/ShawnRG/DataQualityDashboard"
                        sh "git push https://${env.GITHUB_USERNAME}:${env.GITHUB_TOKEN}@github.com/ShawnRG/DataQualityDashboard --tags"
                    }
                }
            }
        }
        stage('Wait for completed build') {
            steps {
//                 when {
//                     expression {
//                         return env.SKIP == 'FALSE'
//                     }
//                 }
                script {
                    remote_sha = sh (
                        script: "git rev-parse origin/main",
                        returnStdout: true
                    ).trim()
                    timeout(time: 1, unit: 'MINUTES') {
                        waitUntil {
                            r_package_sha = sh(
                                script: "curl -X GET ${env.RSTUDIO_PACKAGE_MANAGER_HOST}__api__/repos/4/packages/DataQualityDashboard | python -c \"import sys,json; print json.load(sys.stdin)['remote_sha']\"",
                                returnStdout: true
                            ).trim()
                            return remote_sha == r_package_sha
                        }
                    }
                }
            }

        }

//         stage('Build container') {
//
//         }
    }

}

def getNewVersion(oldVersion) {
    versionParts = oldVersion.split('\\.')
    majorVersionNumber = versionParts[0]
    minorVersionNumber = versionParts[1] as Integer
    return "${majorVersionNumber}.${minorVersionNumber + 1}"
}