
node() {
    try {
        String ANSI_GREEN = "\u001B[32m"
        String ANSI_NORMAL = "\u001B[0m"
        String ANSI_BOLD = "\u001B[1m"
        String ANSI_RED = "\u001B[31m"
        String ANSI_YELLOW = "\u001B[33m"

        ansiColor('xterm') {
            stage('Checkout') {
                cleanWs()
                if (params.github_release_tag == "") {
                    checkout scm
                    commit_hash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    branch_name = sh(script: 'git name-rev --name-only HEAD | rev | cut -d "/" -f1| rev', returnStdout: true).trim()
                    artifact_version = branch_name + "_" + commit_hash
                    println(ANSI_BOLD + ANSI_YELLOW + "github_release_tag not specified, using the latest commit hash: " + commit_hash + ANSI_NORMAL)
                    sh "git clone https://github.com/project-sunbird/sunbird-content-plugins.git plugins"
                    sh "cd plugins && git checkout origin/${branch_name} -b ${branch_name}"
                } else {
                    def scmVars = checkout scm
                    checkout scm: [$class: 'GitSCM', branches: [[name: "refs/tags/${params.github_release_tag}"]], userRemoteConfigs: [[url: scmVars.GIT_URL]]]
                    artifact_version = params.github_release_tag
                    commit_hash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    branch_name = params.github_release_tag.split('_')[0].split('\\.')[0] + "." + params.github_release_tag.split('_')[0].split('\\.')[1]
                    println(ANSI_BOLD + ANSI_YELLOW + "github_release_tag specified, building from github_release_tag: " + params.github_release_tag + ANSI_NORMAL)
                    sh "git clone https://github.com/project-sunbird/sunbird-content-plugins.git plugins"
                    sh """
                        cd plugins
                        checkout_tag=\$(git ls-remote --tags origin $branch_name* | grep -o "$branch_name.*" | sort -V | tail -n1)
                        git checkout tags/\${checkout_tag} -b \${checkout_tag}
                    """
                }
                echo "artifact_version: " + artifact_version

                stage('Build') {
                    sh """
                        export version_number=${branch_name}
                        export build_number=${commit_hash}
                        rm -rf generic-editor
                        node -v
                        npm -v
                        npm install
                        cd app
                        bower cache clean
                        bower install --force
                        cd ..
                        gulp packageCorePlugins
                        npm run plugin-build
                        npm run build
                        #gulp build
                    """
                }
                
                stage('Publish_test_results') {
               cobertura autoUpdateHealth: false, autoUpdateStability: false, coberturaReportFile: 'coverage/PhantomJS*/cobertura-coverage.xml', conditionalCoverageTargets: '70, 0, 0', failUnhealthy: false, failUnstable: false, lineCoverageTargets: '80, 0, 0', maxNumberOfBuilds: 0, methodCoverageTargets: '80, 0, 0', onlyStable: false, sourceEncoding: 'ASCII', zoomCoverageChart: false 
            }
                
                stage('ArchiveArtifacts') {
                    sh """
                        mkdir generic-editor-artifacts
                        cp generic-editor.zip generic-editor-artifacts
                        zip -j  generic-editor-artifacts.zip:${artifact_version}  generic-editor-artifacts/*                      
                    """
                    archiveArtifacts "generic-editor-artifacts.zip:${artifact_version}"
                    sh """echo {\\"artifact_name\\" : \\"generic-editor-artifacts.zip\\", \\"artifact_version\\" : \\"${artifact_version}\\", \\"node_name\\" : \\"${env.NODE_NAME}\\"} > metadata.json"""
                    archiveArtifacts artifacts: 'metadata.json', onlyIfSuccessful: true
                    currentBuild.description = "${artifact_version}"
                }
            }
        }
    }
    catch (err) {
        currentBuild.result = "FAILURE"
        throw err
    }

}
