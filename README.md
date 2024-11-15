# Jenkins Pipeline for Node.js Application Deployment

## Overview
This Jenkins pipeline automates the deployment of a Node.js application by performing the following steps:  
- Pulling source code from a specified Git branch.  
- Installing dependencies.  
- Building the application.  
- Restarting the PM2 service.  
- Sending notifications to a Slack channel at different stages of the pipeline.

---

## Pipeline Details

### 1. **Pipeline Configuration**

#### Agent
- The pipeline is configured to run on an agent with the specified label.

#### Tools
- Node.js is specified as a required tool and must be pre-installed on the agent.

#### Parameters
- **BRANCH_NAME**: A string parameter that allows users to specify the branch name to be built. Defaults to `branch`.

#### Triggers
- **Generic Webhook Trigger**:  
  The pipeline triggers on a webhook with the following settings:
  - A regular expression filter ensures that only changes to the specified branch trigger the pipeline.
  - A token is used to secure the webhook trigger.

---

### 2. **Stages**

#### **Stage 1: Notify Trigger**
- Sends a notification to a Slack channel indicating that the job has been triggered.
- The message format is:


#### **Stage 2: Pulling Source Code**
- Navigates to the specified directory and pulls the source code from the Git repository.
- Steps:
- Checks out the branch specified in the `BRANCH_NAME` parameter.
- Updates the branch using `git pull --rebase origin <branch>`.

#### **Stage 3: Install Dependencies**
- Ensures the environment is prepared by installing Node.js dependencies.  
- Steps:
- Removes the `cache_file/` directory if it exists.
- Installs dependencies using `npm install --legacy-peer-deps`.

#### **Stage 4: Build**
- Builds the application using the defined build script.
- Steps:
- Executes the `npm run build` command.

#### **Stage 5: Restart PM2 Service**
- Restarts the PM2 service to deploy the new build.
- Steps:
- Uses the `pm2 reload` command to restart the application.

---

### 3. **Post-Build Actions**

#### **Notifications**
- A Slack notification is sent upon completion of the pipeline.
- Notification details:
- **Channel**: Configurable Slack channel.
- **Message**:
  ```
  <Build Status>: Job <JOB_NAME> build <BUILD_NUMBER>
  More info at: <BUILD_URL>
  ```
- **Color**: Determined by the build status using the `COLOR_MAP`.

---

## Slack Integration

- The pipeline sends notifications to a Slack channel at:
- The start of the pipeline.
- After the pipeline completes (success, failure, or any status).

---

## Configuration Files

#### **COLOR_MAP**
Used to map build statuses to Slack message colors:
```groovy
def COLOR_MAP = [
  'SUCCESS' : 'good',
  'FAILURE' : 'danger',
  'UNSTABLE': 'warning',
  'ABORTED' : 'warning'
]
