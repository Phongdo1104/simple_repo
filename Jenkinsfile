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
def timestamp = ''
// def prURL=''

pipeline {
    agent any

    environment {
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
        // choice(name: 'ACTION', choices: ['Update only Selected Server', 'Bulk update to all servers'],
        //        description: "Select the action to perform.\n*Please select servers bellow if using [Update only Selected Server]")

        // Checkbox-like parameter to apply for [STG] iijm-st-suggest-api1
        // if you want to apply for other servers, you can add more boolean parameters & edit in booleanParams
        // booleanParam(name: "ubuntu_cli_server", defaultValue: false,description: "[LOCAL] Ubuntu CLI 24.04.2")

        // Choice parameter for the user to select a branch build option
        // choice(name: 'SOURCE_CODE', choices: ['No update source code', 'Update source using BUILD_BRANCH'],description: "please input the BUILD_BRANCH")
        // Text parameter for dynamic environment variables (for update/add)
        // text(name: 'BUILD_BRANCH', defaultValue: 'main',description: "Define branch to checkout\nRunning only select `SOURCE_CODE` as option [Update source using BUILD_BRANCH]")
        // booleanParam(name: "skip_sonar_analyze", defaultValue: false, description: "Skip scanning & analysis via SonarQube")
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
            regexpFilterExpression: 'opened:main',
            regexpFilterText:        '$ACTION:$TARGET_BRANCH',

            // customize your build’s “cause”
            causeString: 'Git PR #$PR_ID'
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
        stage('Preparation') {
            when {
                expression {
                    return env.PR_ID != null
                }
            }
            steps {
                script {
                    def jobBaseName = env.JOB_BASE_NAME;

                    sh """
                        git config --global --add safe.directory /var/lib/jenkins/workspace/${jobBaseName}
                    """
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
                    timestamp = new Date().format("yyyyMMdd_HHmm")
                    filename = "jenkins_credentials_${timestamp}"
                    branch = env.SOURCE_BRANCH
                    prId = env.PR_ID
                    sh """
                        qodo \
                            code-review \
                            --ci \
                            --agent-file path/to/agent.toml --key-value-pairs "target_branch=${branch},severity_threshold=high,focus_areas=security,performance,include_suggestions=true" \
                            > review-output-${timestamp}.json 2> review-debug-${timestamp}.log \
                            || exit 1
                    """
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
    }
}
