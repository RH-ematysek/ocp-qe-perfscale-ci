@Library('flexy') _

// rename build
def userId = currentBuild.rawBuild.getCause(hudson.model.Cause$UserIdCause)?.userId
if (userId) {
  currentBuild.displayName = userId
}

pipeline {
  agent none

  parameters {
        string(name: 'BUILD_NUMBER', defaultValue: '', description: 'Build number of job that has installed the cluster.')
        string(name: 'PROJECT_BASENAME', defaultValue:'logtest', description:'Project name prefix. Will oc delete all projects matching $PROJECT_BASENAME*')
        string(name:'JENKINS_AGENT_LABEL',defaultValue:'oc49 || oc48 || oc47')
        text(name: 'ENV_VARS', defaultValue: '', description:'''<p>
               Enter list of additional (optional) Env Vars you'd want to pass to the script, one pair on each line. <br>
               e.g.<br>
               SOMEVAR1='env-test'<br>
               SOMEVAR2='env2-test'<br>
               ...<br>
               SOMEVARn='envn-test'<br>
               </p>'''
            )
        string(name: 'WORKLOADS_REPO', defaultValue:'https://github.com/RH-ematysek/workloads', description:'You can change this to point to your fork if needed.')
        string(name: 'WORKLOADS_REPO_BRANCH', defaultValue:'logtest_v45_svt', description:'You can change this to point to a branch on your fork if needed.')
    }

  stages {
    stage('Sequential'){
      agent { label params['JENKINS_AGENT_LABEL'] }
      stages{
        stage('Copy artifacts'){
          steps{
            copyArtifacts(
                filter: '',
                fingerprintArtifacts: true,
                projectName: 'ocp-common/Flexy-install',
                selector: specific(params.BUILD_NUMBER),
                target: 'flexy-artifacts'
            )
            script {
              buildinfo = readYaml file: "flexy-artifacts/BUILDINFO.yml"
              currentBuild.displayName = "${currentBuild.displayName}-${params.BUILD_NUMBER}-${currentBuild.number}"
              currentBuild.description = "Copying Artifact from Flexy-install build <a href=\"${buildinfo.buildUrl}\">Flexy-install#${params.BUILD_NUMBER}</a>"
              buildinfo.params.each { env.setProperty(it.key, it.value) }
            }
          }
        }
        stage('Checkout workload repo'){
          steps{
            git branch: params.WORKLOADS_REPO_BRANCH, url: params.WORKLOADS_REPO
          }
        }
        stage('Run cleanup'){
          steps{
            ansiColor('xterm') {
              sh label: '', script: '''
              # Get ENV VARS Supplied by the user to this job and store in .env_override
              echo "$ENV_VARS" > .env_override
              # Export those env vars so they could be used by CI Job
              set -a && source .env_override && set +a
              mkdir -p ~/.kube
              cp $WORKSPACE/flexy-artifacts/workdir/install-dir/auth/kubeconfig ~/.kube/config

              echo -e "[orchestration]\nlocalhost ansible_connection=local" > inventory
              cat inventory
              TAGS="delete_indices,delete_indices_req,clear_buffers"
              if oc get projects | grep -qE "^$PROJECT_BASENAME.+Active"; then
                TAGS+=",cleanup"
                echo "Projects found to delete:"
                oc get projects | grep "^$PROJECT_BASENAME"
              else
                echo "No projects found to cleanup, skipping project deletion step"
              fi
              ORCHESTRATION_USER="$(whoami)" PROJECT_BASENAME=$PROJECT_BASENAME ansible-playbook -v -i inventory workloads/logging.yml -v --tags $TAGS
              '''
            }
          }
        }
      }
    }

  }
}

