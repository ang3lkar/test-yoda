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
            channel: process.env.YODA_SLACK_CHANNEL,
            color: "danger",
            message: "${JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)\n```${BRANCH}```${isAbortException ? '' : '\n```' + caughtError + '```'}"
          ])
          if (caughtError) {
           throw caughtError; // rethrow so that the build fails
          }
          break;
        default:
          slackit([
            channel: process.env.YODA_SLACK_CHANNEL,
            color: "good",
            message: "${JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)\n```${BASIS_BRANCH} - ${BRANCH}()\nhttps://docs.google.com/spreadsheets/d/${YODA_SHEET_ID}```"
          ])
      }
    }
  }
}

def killRails() {
  sh "if (lsof -i -P | grep LISTEN | grep '*:3000' > pid.txt); then kill -9 \$(cat pid.txt | head -1 | awk '{print \$2}'); else echo 'Rails is not running'; fi"
}

def killSolr() {
  sh "if (lsof -i -P | grep LISTEN | grep '*:8982' > pid.txt); then kill -9 \$(cat pid.txt | head -1 | awk '{print \$2}'); else echo 'Solr is not running'; fi"
}

def slackit(params) {
  if (!env.SKIP_SLACK) {
    slackSend(params)
  }
}

def sanitizeJobName(jobName) {
  return jobName.replaceAll('%2F', '/');
}
