import groovy.json.JsonSlurperClassic

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
          INDICATIVE_RESULTS = "139882596 3124 2921"
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
          resultsMap = INDICATIVE_RESULTS.tokenize(' ')
          sheetId = resultsMap[0]
          basisBranchTime = resultsMap[1]
          branchTime = resultsMap[2]

          slackit([
            channel: YODA_SLACK_CHANNEL,
            color: "good",
            message: """
                    |${JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)\n 
                    |```
                    |${BASIS_BRANCH} - ${BRANCH} (${COMMIT_HASH})
                    |
                    |DASHBOARD
                    |${BASIS_BRANCH}:\t ${basisBranchTime}ms
                    |${BRANCH}:\t ${branchTime}ms
                    |```
                    |\n
                    |Results available at:\nhttps://docs.google.com/spreadsheets/d/${YODA_SHEET_ID}#gid=${sheetId}
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
