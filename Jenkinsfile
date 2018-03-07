String GIT_LOG;
String GIT_LOCAL_BRANCH;

def caughtError;

node(label: 'Angelos-Slave') {
  currentBuild.result = "SUCCESS"

  ansiColor('xterm') {
    try {

      stage('Checkout') {
         checkout scm
      }

      String PATH = "PATH=$PATH:/usr/local/bin"; // this is to find the npm command

      lock(resource: 'mobile-web-performance-lock') {
        environment { 
          COMMIT_HASH = sh "git ls-remote --heads git@github.com:Workable/workable.git | grep \$BRANCH | awk '{print \$1}'"
          sh "echo \$COMMIT_HASH"
        }
      }

    } catch (err) {
      caughtError = err;
      currentBuild.result = "FAILURE"
    } finally {
      switch(currentBuild.result) {
        case "FAILURE":
          slackit([
            channel: YODA_SLACK_CHANNEL,
            color: "danger",
            message: "${JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)\n```${BRANCH}```${isAbortException ? '' : '\n```' + caughtError + '```'}"
          ])
          if (caughtError) {
           throw caughtError; // rethrow so that the build fails
          }
          break;
        default:
          slackit([
            channel: YODA_SLACK_CHANNEL,
            color: "good",
            message: "${JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)\n```${BASIS_BRANCH} - ${BRANCH}()\n\nResults available at:\nhttps://docs.google.com/spreadsheets/d/${YODA_SHEET_ID}```"
          ])
      }
    }
  }
}

def slackit(params) {
  if (!env.SKIP_SLACK) {
    slackSend(params)
  }
}

def sanitizeJobName(jobName) {
  return jobName.replaceAll('%2F', '/');
}
