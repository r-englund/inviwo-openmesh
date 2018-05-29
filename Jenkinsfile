@NonCPS
def getChangeString() {
    MAX_MSG_LEN = 100
    def changeString = ""

    echo "Gathering SCM changes"
    def changeLogSets = currentBuild.rawBuild.changeSets
    for (int i = 0; i < changeLogSets.size(); i++) {
        def entries = changeLogSets[i].items
        for (int j = 0; j < entries.length; j++) {
            def entry = entries[j]
            truncated_msg = entry.msg.take(MAX_MSG_LEN)
            changeString += "${new Date(entry.timestamp).format("yyyy-MM-dd HH:mm:ss")} "
            changeString += "[${entry.commitId.take(8)}] ${entry.author}: ${truncated_msg}\n"
        }
    }

    if (!changeString) {
        changeString = " - No new changes"
    }
    return changeString
}

def nicelog(env = [], fun) {
    withEnv(['TERM=xterm'] + env) {
        ansiColor {
            timestamps {
                fun()
            }
        }
    }
}

node {
    properties([
        parameters([
            booleanParam(
                defaultValue: false, 
                description: 'Do a clean build', 
                name: 'Clean Build'
            ),
            choice(
                // The first will be default
                choices: "Release\nDebug\nMinSizeRel\nRelWithDebInfo\n", 
                description: 'Select build configuration', 
                name: 'Build Type'
            ),
            [$class: 'GitParameterDefinition', 
                branch: '', 
                branchFilter: '.*', 
                defaultValue: 'master', 
                description: 'Inviwo branch to use in build', 
                name: 'Inviwo Branch', 
                quickFilterEnabled: true, 
                selectedValue: 'DEFAULT', 
                sortMode: 'NONE', 
                tagFilter: '*', 
                type: 'PT_BRANCH', 
                useRepository: '.*inviwo.*']
            ]),
        [$class: 'GithubProjectProperty', 
            displayName: 'inviwo-openmesh', 
            projectUrlStr: 'https://github.com/r-englund/inviwo-openmesh/'],
        [$class: 'RebuildSettings', 
            autoRebuild: false, 
            rebuildDisabled: false], 
            pipelineTriggers([[$class: 'GitHubPushTrigger']])
        ])
    
    try {
        stage('Fetch') { 
                echo "Building inviwo-openmesh. Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
                dir('inviwo-openmesh/modules/openmesh') {
                    checkout scm
                    sh 'git submodule sync'
                    sh 'git submodule update --init'
                }
                dir('inviwo') {
                    echo "Inviwo branch: ${params['Inviwo Branch']}"
                    git([branches: '*/master', 
                         credentialsId: 'bc26c365-0b51-47cf-8af0-4043bd3e054c', 
                        url: 'https://github.com/inviwo/inviwo.git'])
                    sh 'git submodule update --init'
                }
            }
        stage('Build') {
            if (params['Clean Build']) {
                echo "Clean build, removing build folder"
                sh "rm -r build"
            }
            dir('build') {
                nicelog(['CC=/usr/bin/gcc-5', 'CXX=/usr/bin/g++-5']) {
                    sh """
                        
                        ccache -z # reset ccache statistics
                        # tell ccache where the project root is
                        export CPATH=`pwd`
                        export CCACHE_BASEDIR=`readlink -f \${CPATH}/..`

                        cmake -G \"Ninja\" -LA \
                              -DCMAKE_BUILD_TYPE=${params['Build Type']} \
                              -DCMAKE_PREFIX_PATH=/opt/Qt/5.6/gcc_64 \
                              -DIVW_CMAKE_DEBUG=ON \
                              -DIVW_EXTERNAL_MODULES=${env.WORKSPACE}/inviwo-openmesh/modules \
                              -DIVW_MODULE_POSTPROCESSING=OFF \
                              -DIVW_MODULE_EIGENUTILS=OFF \
                              -DIVW_MODULE_BRUSHINGANDLINKING=OFF \
                              -DIVW_MODULE_VECTORFIELDVISUALIZATION=OFF \
                              -DIVW_MODULE_NIFTI=OFF \
                              -DIVW_MODULE_PVM=OFF \
                              -DIVW_MODULE_FONTRENDERING=OFF \
                              -DIVW_MODULE_BASECL=OFF \
                              -DIVW_MODULE_OPENCL=OFF \
                              -DIVW_MODULE_GLFW=ON \
                              -DIVW_MODULE_OPENMESH=ON \
                              -DIVW_TINY_GLFW_APPLICATION=OFF \
                              -DIVW_TINY_QT_APPLICATION=OFF \
                              -DIVW_UNITTESTS=ON \
                              -DIVW_UNITTESTS_RUN_ON_BUILD=OFF \
                              -DIVW_INTEGRATION_TESTS=ON \
                              -DIVW_MODULE_TENSORFIELDBASE=OFF \
                              ../inviwo
                        
                        ninja

                        ccache -s # print ccache statistics
                    """
                }
            }
        }

        stage('Unit tests') {
            dir('build/bin') {
                nicelog {
                    sh '''
                        export DISPLAY=:0
                        rc=0
                        for unittest in inviwo-unittests-*
                            do echo ==================================
                            echo Running: ${unittest}
                            ./${unittest} || rc=$?
                        done
                        exit ${rc}
                    '''
                }
            }
        }

        stage('Integration tests') {
            dir('build/bin') {
                nicelog {
                    sh '''
                        export DISPLAY=:0
                        ./inviwo-integrationtests
                    '''
                }
            }
        }

        try {
            stage('Regression tests') {
                dir('regress') {
                    nicelog {
                        sh """
                            export DISPLAY=:0
                            python3 ../inviwo/tools/regression.py \
                                    --inviwo ../build/bin/inviwo \
                                    --header ${env.JENKINS_HOME}/inviwo-config/header.html \
                                    --output . \
                                    --repos ../inviwo-openmesh \
                                    --include openmesh
                        """
                    }
                }
            }
        } catch (e) {
            // Mark as unstable, if we mark as failed, the report will not be published.
            currentBuild.result = 'UNSTABLE'
        }
        stage('Publish') {
            publishHTML([
                allowMissing: true,
                alwaysLinkToLastBuild: true,
                keepAll: false,
                reportDir: 'regress',
                reportFiles: 'report.html',
                reportName: 'Regression Report'
            ])
        }
        currentBuild.result = 'SUCCESS'
    } catch (e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        stage('Slack') {
            echo "result: ${currentBuild.result}"
            def res2color = ['SUCCESS' : 'good', 'UNSTABLE' : 'warning' , 'FAILURE' : 'danger' ]
            def color = res2color.containsKey(currentBuild.result) ? res2color[currentBuild.result] : 'warning'
            slackSend(
                color: color, 
                channel: "#inviwo-openmesh", 
                message: "Research dev branch: ${env.BRANCH_NAME}\n" + \
                         "Status: ${currentBuild.result}\n" + \
                         "Job: ${env.BUILD_URL}\n" + \
                         "Regression: ${env.JOB_URL}Regression_Report/\n"+ \
                         "Changes: " + getChangeString() 
            )
        }
    }
}
