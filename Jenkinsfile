import groovy.json.JsonSlurper

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
        stage('foo') {
          COMMIT_HASH = sh (
            script: "git ls-remote --heads git@github.com:Workable/workable.git | grep \$BRANCH\$ | awk '{print \$1}'",
            returnStdout: true).trim()

          def jsonSlurper = new JsonSlurper()
          def object = jsonSlurper.parseText('{"sheetId":139882596, "basisBranchDashboardTime":3124, "branchDashboardTime":2921}');
          INDICATIVE_RESULTS = object
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
            message: """
                    |${JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)\n 
                    |```
                    |${BASIS_BRANCH} - ${BRANCH} (${COMMIT_HASH})
                    |
                    |DASHBOARD
                    |${BASIS_BRANCH}:\t ${INDICATIVE_RESULTS.basisBranchDashboardTime}ms
                    |${BRANCH}:\t ${INDICATIVE_RESULTS.branchDashboardTime}ms
                    |```
                    |\n
                    |Results available at:\nhttps://docs.google.com/spreadsheets/d/${YODA_SHEET_ID} 
                    """.stripMargin()
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
