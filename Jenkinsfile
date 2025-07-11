def servers = [
    [serverIp: '192.168.1.218', serverName: '[LOCAL] Ubuntu CLI 24.04.2', selected: false, removedCredentialFiles:false],
]

def BITO_SERVER = '192.168.1.218'
def branch=''
def selectedServers=[]
def hasServerSelected=false
def selectedServerNameString =''
def userConfirm = true
def filename=''
def pathCredential=''
def filePathCredential=''
def removeCredentialFiles=false
def USER_SSH=''
def SKIP_ANALYZE=false
// def prURL=''

pipeline {
    agent any

    environment {
        projectName = 'simple_repo'
        // Define the env file location as a global variable
        projectPath = '/home/new_laravel_test/test-laravel/laravel'
        GIT_CREDENTIALS_ID = 'jenkins-test-github'
        GIT_URL = 'github.com'
        GIT_CREDENTIALS_PATH_FOLDER = '/tmp/jenkin_credentials'
        SSH_AGENT_ACCOUNT = 'ssh-remote-ubuntu'
        // SONAR_SCANNER_HOME = tool 'SonarScannerStage'
        SONAR_TOKEN_ID = 'sonar-token-id'
        SONAR_SERVER_ID = "SonarQubeServer"
        BITO_AI_PATH = '/home/bito-ai/CodeReviewAgent/cra-scripts'
        GITHUB_TOKEN = credentials('github-token')
        QODO_API_KEY = credentials('qodo-api-token')
    }

    parameters {
        // Choice parameter for the user to select an action
        choice(name: 'ACTION', choices: ['Update only Selected Server', 'Bulk update to all servers'],
               description: "Select the action to perform.\n*Please select servers bellow if using [Update only Selected Server]")

        // Checkbox-like parameter to apply for [STG] iijm-st-suggest-api1
        // if you want to apply for other servers, you can add more boolean parameters & edit in booleanParams
        booleanParam(name: "ubuntu_cli_server", defaultValue: false,description: "[LOCAL] Ubuntu CLI 24.04.2")

        // Choice parameter for the user to select a branch build option
        choice(name: 'SOURCE_CODE', choices: ['No update source code', 'Update source using BUILD_BRANCH'],description: "please input the BUILD_BRANCH")
        // Text parameter for dynamic environment variables (for update/add)
        text(name: 'BUILD_BRANCH', defaultValue: 'main',description: "Define branch to checkout\nRunning only select `SOURCE_CODE` as option [Update source using BUILD_BRANCH]")
        booleanParam(name: "skip_sonar_analyze", defaultValue: false, description: "Skip scanning & analysis via SonarQube")
        // text(name: 'PR_PATH', defaultValue: '', description: 'Pull Request Path')
    }

    triggers {
        GenericTrigger(
            token:         'TOKEN_HERE',
            genericVariables: [
                [ key: 'PR_ID',         value: '$.pull_request.number' ],
                [ key: 'SOURCE_BRANCH', value: '$.pull_request.head.ref' ],
                [ key: 'TARGET_BRANCH', value: '$.pull_request.base.ref' ],
                [ key: 'ACTION',        value: '$.action'],
                [ key: 'MERGE_STATUS',  value: '$.pull_request.merged']
            ],
            // for debugging:
            printPostContent:       true,
            printContributedVariables: true,
            // regexpFilterExpression: 'opened',
            // regexpFilterText:        '$ACTION',
            regexpFilterExpression: 'opened',
            regexpFilterText:        '$ACTION',

            // customize your build’s “cause”
            causeString: 'GitHub PR #$PR_ID'
        )
    }

    stages {
        stage('Show PR info') {
            when {
                expression {
                    return env.PR_ID != null;
                }
            }
            steps {
                script {
                    if (params.skip_sonar_analyze) {
                        SKIP_ANALYZE = true
                    }
                    def pdId = """
                        Pull Request #${env.PR_ID}
                    """
                    def sourceBranch = """
                        Source Branch: ${env.SOURCE_BRANCH}
                    """
                    def targetBranch = """
                        Target Branch: ${env.TARGET_BRANCH}
                    """
                    def actionStatus = """
                        Action: ${env.ACTION}
                    """
                    def mergeStatus = """
                        Merged status: ${MERGE_STATUS}
                    """
                    def skipAnalyzeFlg = """
                        Skip Analyze Flag: #${SKIP_ANALYZE}
                    """
                    def userInput = input(
                        message: """${pdId} ${sourceBranch} ${targetBranch} ${actionStatus} ${mergeStatus} ${skipAnalyzeFlg}""",
                    )
                }
            }
        }
        // stage('Initialize Docker') {
        //     when {
        //         expression {
        //             return env.PR_ID != null
        //         }
        //     }
        //     steps {
        //         script {
        //             def dockerHome = tool 'DockerTool'
        //             env.PATH = "${dockerHome}/bin:${env.PATH}"
        //         }
        //     }
        // }
        stage('Preparation') {
            when {
                expression {
                    return env.PR_ID != null
                }
            }
            steps {
                script {
                    def timestamp = new Date().format("yyyyMMdd_HHmm")
                    filename = "jenkins_credentials_${timestamp}"

                    pathCredential="""${GIT_CREDENTIALS_PATH_FOLDER}/${filename}"""
                    filePathCredential="""${pathCredential}/.${filename}"""

                    env.PATH_CREDENTIAL=filePathCredential
                    env.FILE_PATH_CREDENTIAL=filePathCredential

                    echo 'make temp folder'
                    sh "mkdir -p ${pathCredential}"

                    withCredentials([usernamePassword(credentialsId: "${GIT_CREDENTIALS_ID}", usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        sh '''
                            echo "https://${GIT_USERNAME}:${GIT_PASSWORD}@${GIT_URL}" >> $FILE_PATH_CREDENTIAL
                        '''

                        sh "cat $FILE_PATH_CREDENTIAL"

                        sh '''
                            git config --global --add safe.directory /var/lib/jenkins/workspace/${projectName}
                        '''
                    }
                }
            }
        }
        stage('Code Review') {
            when {
                expression {
                    return env.PR_ID != null;
                }
            }
            steps {
                script {
                    branch = env.SOURCE_BRANCH
                    prId = env.PR_ID
                    // run docker -v first to check
                    docker.image('qodoai/command:latest').inside {
                        sh '''
                              qodo --ci code-review --agent-file path/to/agent.toml --key-value-pairs "target_branch=${branch},severity_threshold=high,focus_areas=security,performance,include_suggestions=true"
                        '''
                    }
                }
            }
        }
        stage('test') {
            steps {
                script {
                    sh "echo 'hello world'"
                }
            }
        }

        // stage('Map selected server') {
        //     steps {
        //         script {
        //             // Create a list of the boolean params
        //             def booleanParams = [
        //                 params.ubuntu_cli_server,
        //             ]

        //             if (params.ACTION == 'Bulk update to all servers') {
        //                 // Map the booleans to the "selected" attribute of the array objects
        //                 servers.eachWithIndex { item, index ->
        //                     item.selected = true
        //                 }
        //             } else {
        //                 // Map the booleans to the "selected" attribute of the array objects
        //                 servers.eachWithIndex { item, index ->
        //                     item.selected = booleanParams[index]
        //                 }
        //             }

        //             // Filter servers selected
        //             selectedServers = servers.findAll { server ->
        //                 server.selected
        //             }

        //             if (selectedServers) {
        //                 hasServerSelected = true
        //             }

        //             if(params.SOURCE_CODE == 'Update source using BUILD_BRANCH'){
        //                 branch = params.BUILD_BRANCH ?: 'main'
        //             }

        //             if (params.skip_sonar_analyze) {
        //                 SKIP_ANALYZE = true
        //             }

        //             if (!params.PR_PATH.isEmpty()) {
        //                 prURL = params.PR_PATH;
        //             }

        //             // Echo the selected servers in one line
        //             if (selectedServers) {
        //                 hasServerSelected = true
        //                 def selectedServerNames = selectedServers.findAll { it.selected }.collect { it.serverName }
        //                 selectedServerNameString = selectedServerNames.join(', ')
        //                 echo """Deploying to server: ${selectedServerNameString}"""
        //             } else {
        //                 echo "No servers selected for deployment."
        //             }

        //             withCredentials([sshUserPrivateKey(credentialsId: "${SSH_AGENT_ACCOUNT}", keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USER')]) {
        //                 USER_SSH = "${SSH_USER}"
        //             }
        //         }
        //     }
        // }

        /** run these git command to merge
            git fetch --all -p
            git pull origin main
            git merge origin/<branch_name>
            git commit -m "Merge pull request #<pr_id> from username/branch"
            git push origin main
         */


        // stage('Deployer Confirmation Setting') {
        //     when {
        //         expression { hasServerSelected == true }
        //     }
        //     steps {
        //         script {
        //             // Prompt the user for confirmation
        //             def configSetting = """
        //             Do you want to proceed with the config bellow?

        //             - Selected servers :
        //             ${selectedServerNameString ?: "NO server selected"}
        //             """

        //             def branchSetting = branch.isEmpty() ?
        //             ''
        //             :
        //             """
        //             ⁜ Update source code with branch: ${branch}
        //             """

        //             def sonarQubeSkipFlg = params.SKIP_ANALYZE ? "[x] Yes | [ ] No" : "[ ] Yes | [x] No"

        //             def sonarQubeSetting =
        //             """
        //             ⁜ Skip SonarQube analysis? ${sonarQubeSkipFlg}
        //             """

        //             def prURLSettings =
        //             """
        //             ⁜ Pull Request path: ${prURL}
        //             """

        //             def userInput = input(
        //                 message: """${configSetting} ${branchSetting} ${sonarQubeSetting} ${prURLSettings}""",
        //             )
        //         }
        //     }
        // }

        // stage('Checkout SCM') {
        //     steps {
        //         checkout scm
        //     }
        // }

        // stage('SonarQube Analysis') {
        //     when {
        //         expression { SKIP_ANALYZE == false }
        //     }
        //     steps {
        //         withCredentials([string(credentialsId: "${SONAR_TOKEN_ID}", variable: "SONAR_TOKEN")]) {
        //             withSonarQubeEnv("SonarQubeServer") {
        //                 sh """
        //                     ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
        //                         -Dsonar.projectKey=jenkins-sonarqube-laravel \
        //                         -Dsonar.sources=. \
        //                         -Dsonar.exclusions=composer.json,composer.lock,vendor/**/*
        //                 """
        //             }
        //         }
        //     }
        // }

        // stage('Wait for Quality gate And Approval') {
        //     when {
        //         expression { SKIP_ANALYZE == false }
        //     }
        //     steps {
        //         script {
        //             timeout(time: 1, unit: 'HOURS') {
        //                 // Wait for SonarQube to compute the quality gate
        //                 def qg = waitForQualityGate(abortPipeline: false)
        //                 echo "Quality Gate Status: ${qg.status}"

        //                 // Prompt user depending on result
        //                 if (qg.status != 'OK') {
        //                     input message: "Quality Gate is *${qg.status}*. Approve to proceed 👇", ok: "Proceed"
        //                 } else {
        //                     input message: "Quality Gate passed! Continue to deploy?", ok: "Deploy"
        //                 }
        //             }
        //         }
        //     }
        // }

        // stage('Deploying on Servers') {
        //     when {
        //         expression { hasServerSelected == true }
        //     }
        //     steps {
        //         script{
        //             // Get the current datetime to append to the filename
        //             def timestamp = new Date().format("yyyyMMdd_HHmm")
        //             filename = "jenkins_credentials_${timestamp}"

        //             pathCredential="""${GIT_CREDENTIALS_PATH_FOLDER}/${filename}"""
        //             filePathCredential="""${pathCredential}/.${filename}"""

        //             env.PATH_CREDENTIAL=filePathCredential
        //             env.FILE_PATH_CREDENTIAL=filePathCredential


        //             echo 'make temp folder'
        //             sh "mkdir -p ${pathCredential}"
        //             if(branch != ''){
        //                 withCredentials([usernamePassword(credentialsId: "${GIT_CREDENTIALS_ID}", usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
        //                     // Pass credentials as environment variables, avoid logging sensitive data
        //                     sh '''
        //                         echo "https://${GIT_USERNAME}:${GIT_PASSWORD}@${GIT_URL}" >> $FILE_PATH_CREDENTIAL
        //                     '''

        //                     sh "cat $FILE_PATH_CREDENTIAL"
        //                 }
        //             }

        //             // Create a map to hold the parallel stages
        //             def wparallelStages = selectedServers.collectEntries { server ->
        //                 return ["${server.serverName}" : {
        //                     if (selectedServers.size() > 1) {
        //                         stage("Confirm to running deploy on ${server.serverName}") {
        //                             def userInput = input(
        //                                 message: """Do you want to proceed with the deploy on ${server.serverName}""",
        //                             )
        //                         }
        //                     }

        //                     if(branch != ''){
        //                         stage("Update selected source on ${server.serverName}") {
        //                             withCredentials([usernamePassword(credentialsId: "${GIT_CREDENTIALS_ID}", usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
        //                                 sshagent(["${SSH_AGENT_ACCOUNT}"]) { // Use the ID of your credential
        //                                     script {
        //                                         // Execute commands on Server 1 to update/add variables
        //                                         def currentBranch = sh(script:"""
        //                                             ssh -o StrictHostKeyChecking=no ${USER_SSH}@${server.serverIp} << EOF

        //                                             # Create a backup of the existing .env file with the current date
        //                                             cd "${projectPath}"

        //                                             git rev-parse --abbrev-ref HEAD
        //                                         """, returnStdout: true).trim()

        //                                         def result = 0
        //                                         def cmd = ""
        //                                         if(currentBranch == branch){
        //                                             echo "Updating ${branch} on ${server.serverName} with git pull"
        //                                             cmd = """
        //                                                 # Update or add environment variables
        //                                                 pwd
        //                                                 echo "Pull source"
        //                                                 sudo git pull  || exit 1;
        //                                                 sudo docker compose exec -T laravel_app php artisan optimize:clear
        //                                             """
        //                                         } else {
        //                                             echo "Updating branch on ${server.serverName} from ${currentBranch} to ${branch} with git checkout ${branch}"
        //                                             cmd =  """
        //                                                 echo 'Git fetch'
        //                                                 sudo git fetch || exit 1

        //                                                 echo 'Check out branch ${branch}'
        //                                                 sudo git checkout ${branch}  || exit 1

        //                                                 echo 'Pull on remote ${branch}'
        //                                                 sudo git pull || exit 1
        //                                                 sudo docker compose exec -T laravel_app php artisan optimize:clear
        //                                             """
        //                                         }
        //                                         sh """
        //                                             ssh -o StrictHostKeyChecking=no ${USER_SSH}@${server.serverIp} '
        //                                             echo "=== STARTING DEPLOY ===";
        //                                             cd ${projectPath} || exit 1;

        //                                             mkdir -p \$(dirname ${filePathCredential});
        //                                             echo "https://${GIT_USERNAME}:${GIT_PASSWORD}@${GIT_URL}" > ${filePathCredential};

        //                                             sudo git config --global credential.helper "store --file=${filePathCredential}" || exit 1;

        //                                             ${cmd}
        //                                             echo "Deleting credential file: ${filePathCredential}";
        //                                             rm -f ${filePathCredential};
        //                                             sudo git config --global --unset credential.helper || exit 1;
        //                                             '
        //                                         """
        //                                     }
        //                                 }
        //                             }
        //                         }
        //                     }
        //                 }]
        //             }
        //             // Run all tasks in parallel
        //             parallel wparallelStages
        //         }
        //     }
        // }
    }

    // post {
    //     always {
    //         script {
    //             if(pathCredential){

    //                 echo "Remove ${pathCredential} on current Jenkins server"
    //                 if(pathCredential){
    //                     echo "remove ${pathCredential}"
    //                     sh """rm -rf ${pathCredential}"""
    //                 }

    //                 // Create a map to hold the parallel stages
    //                 def parallelStages = selectedServers.collectEntries { server ->
    //                     ["${server.serverName}" : {
    //                         stage("REMOVE jenkins_credentials on ${server.serverName}") {
    //                             sshagent(["${SSH_AGENT_ACCOUNT}"]) { // Use the ID of your credential
    //                                 script {
    //                                     if(server.removedCredentialFiles==true){
    //                                         echo "SKIP ON ${pathCredential} on ${server.serverIp}"
    //                                     }else{
    //                                         sh """
    //                                             ssh ${USER_SSH}@${server.serverIp} << EOF

    //                                             #disable the history command
    //                                             set +o history

    //                                             cd "${projectPath}"

    //                                             echo "remove the credential in store file"
    //                                             rm -rf ${pathCredential} || exit 1

    //                                             echo 'Set the credential to none'
    //                                             sudo git config --unset credential.helper

    //                                             #re-enable the history command
    //                                             set -o history
    //                                         """
    //                                     }
    //                                 }
    //                             }
    //                         }
    //                     }]
    //                 }
    //                 // Run all tasks in parallel
    //                 parallel parallelStages
    //             }
    //         }
    //     }
    //     success {
    //         echo 'Deployment was successful!'
    //     }
    //     failure {
    //         echo 'There was a problem with the deployment.'
    //     }
    // }
}
