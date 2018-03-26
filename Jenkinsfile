import groovy.json.JsonSlurperClassic

String GIT_LOG;
String GIT_LOCAL_BRANCH;

def caughtError;

node(label: 'Mobile Builder 2') {
  currentBuild.result = "SUCCESS"

  ansiColor('xterm') {
    try {

      stage('Checkout') {
         checkout scm
      }

      String PATH = "PATH=$PATH:/usr/local/bin"; // this is to find the npm command

      String LOCK_SCOPE = env.NODE_NAME.replace(/ /, '-')
      lock(resource: "mobile-web-performance-lock-$LOCK_SCOPE") {
        stage('foo') {
          COMMIT_HASH = sh (
            script: "git ls-remote --heads https://github.com/Workable/workable.git | grep \$BRANCH\$ | awk '{print \$1}'",
            returnStdout: true).trim()
          INDICATIVE_RESULTS = "139882596 3124 2921"
          echo "$BRANCH:$COMMIT_HASH"
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
          foo = relativeResult(basisBranchTime.toInteger(), branchTime.toInteger())

          slackit([
            channel: YODA_SLACK_CHANNEL,
            color: "good",
            message: """
                    |${JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)\n
                    |```
                    |${BASIS_BRANCH} - ${BRANCH} (${COMMIT_HASH})
                    |
                    |DASHBOARD
                    |${BASIS_BRANCH}:\t  ${basisBranchTime}ms
                    |${BRANCH}:\t ${branchTime}ms - ${foo}
                    |```
                    |\n
                    |Results available at:\nhttps://docs.google.com/spreadsheets/d/${YODA_SHEET_ID}#gid=${sheetId}
                    """.stripMargin()
          ])
      }
    }
  }
}

def relativeResult(int previous, int after) {
  if (previous > after) {
    result = 'faster'
    fraction = after / previous
  } else {
    result = 'slower'
    fraction = previous / after
  }
  // percentage = Math.round(fraction * 100) / 100
  return "${fraction.toString().subString(0, 3)}x ${result}"
}

def slackit(params) {
  if (!env.SKIP_SLACK) {
    slackSend(params)
  }
}