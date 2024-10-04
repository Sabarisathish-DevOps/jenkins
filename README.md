JENKINS CI/CD PIPELINE(detailing):

        def COLOR_MAP = [
        'SUCCESS' : 'good',
        'FAILURE' : 'danger',
        'UNSTABLE': 'warning',
        'ABORTED': 'warning'
    ]

*) Defining the color map for the notification, whether the job build success or not.

            agent {
        label '******'
      } 

*) Agent section defines the node(server). It is used to select the node, where the job can run.

        parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'branch', description: 'Enter the branch name')
      }

*) The string parameter is used to define a string value before the pipeline runs. In this case, i used to define my github branch.

        triggers {
        GenericTrigger(
            genericVariables: [
                [key: 'ref', value: '$.ref']
            ],
            causeString: 'Triggered on $ref',
            token: '******',
            printContributedVariables: true,
            regexpFilterText: '$ref',
            regexpFilterExpression: '^(refs/heads/branch)$'
        )
    }

*) triggers block is used to trigger the job when a push event is made to the mention branch.
   In this case, the GenericTrigger is used, it will check the incoming payload from the webhook and filters the key and value if it is matchs the value that were given, it will trigger the job.
   Giving token matchs the job and the repository.
   The webhook url that has to be updated in the github repository
   https://jenkins_url/generic-webhook-trigger/invoke?token=*******

            stage('Notify Trigger') {
            steps {
                script {
                    slackSend channel:'#deployment',
                        color: 'good',
                        message: "*Job ${env.JOB_NAME}* has been triggered."
                }
            }
        }

*) Notify Trigger this is step is used to send notification in the slack, when the job has triggered.

        stage('Pulling source code') {
            steps {
                dir('path of the directory') {
                    git branch: params.BRANCH_NAME, credentialsId: 'GIT_SSH', url: 'repository url'
                    sh 'git pull --rebase origin branch'
                }
            }
        }
    
*) Pulling source code stage is used to pull the source code in the desired directory. If it is a private repository the credientail must be given in the jenkins credentials.

        stage('Install Dependencies') {
            steps {
                dir('path of the directory') {
                    // Install dependencies using npm
                    sh '''
                    if [ -d "node_modules" ]; then
                        echo "Removing node_modules directory..."
                        rm -r node_modules/
                    else
                        echo "node_modules directory does not exist, skipping..."
                    fi
                    '''

                    // Check if package-lock.json exists, then remove it
                    sh '''
                    if [ -f "package-lock.json" ]; then
                        echo "Removing package-lock.json..."
                        rm package-lock.json
                    else
                        echo "package-lock.json file does not exist, skipping..."
                    fi
                    '''
                    sh 'npm i'
                }
            }
        }

*) Installing dependencies, this stage is used to install dependencies for the appilcation. It will check for the node_modules, dist, package-lock.json in the working directory, if its there it will remove the folders, if its not there it will skip the step and install dependencies.

        stage('Build') {
            steps {
                dir('path of the directory') {
                    // Build your code with npm (replace 'build' with your actual build script if it's different)
                    sh 'npm run build'
                }
            }
        }

*) Build stage, this stage is used to build the code in the working directory.

        post {
        always {
            echo 'Slack Notifications.'
            slackSend channel:'#channelname',
                color:  COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n more info at: ${env.BUILD_URL}"
        }
    }

*) The post block, when the stages block are done the post block triggered. In this case, it will send the notification to the slac whether the build is success or failer.