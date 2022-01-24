@Library('flexy') _

// rename build
def userId = currentBuild.rawBuild.getCause(hudson.model.Cause$UserIdCause)?.userId
if (userId) {
  currentBuild.displayName = userId
}

pipeline {
  agent none

  parameters {
        // Build cluster
        string(name: 'BUILD_NUMBER', defaultValue: '---', description: 'Leave blank if building cluster as part of e2e. Build number of job that has installed the cluster.')
        string(name: 'INSTANCE_NAME_PREFIX', defaultValue: 'logtest', description: '')
        string(name: 'VARIABLES_LOCATION', defaultValue: 'private-templates/functionality-testing/aos-4_10/ipi-on-aws/versioned-installer', description: '')
        string(name: 'INSTALLER_PAYLOAD_IMAGE', defaultValue: 'registry.ci.openshift.org/ocp/release:4.10.0-fc.2', description: '')
        // Prep cluster
        booleanParam(name: 'SCALE_MACHINESETS', defaultValue: true, description: 'Scale worker machinesets to 0, update instance type as specified, then scale up. machineset A is scaled for elasticsearch, B is scaled for fluentd, and all other machinesets are scaled down to 0')
        string(name: 'ELS_INSTANCE_TYPE', defaultValue: 'm6i.2xlarge', description: '')
        string(name: 'FLUENTD_INSTANCE_TYPE', defaultValue: 'm6i.xlarge', description: '')
        string(name: 'NUM_FLUENTD_NODES', defaultValue: '5', description: '')
        booleanParam(name: 'DEPLOY_LOGGING', defaultValue: true, description: 'Deploy cluster loging operator and elasticsearch operator')
        string(name: 'CLO_BRANCH', defaultValue: 'release-5.3', description: 'Branch to deploy Cluster Logging Operator and Elasticsearch Operator from. See https://github.com/openshift/cluster-logging-operator')
        booleanParam(name: 'CREATE_CLO_INSTANCE', defaultValue: true, description: 'Create CLO instance with ELS 4 cpu, 16G RAM, and 200G gp2 ssd storage.')

        // Global
        string(name: 'JENKINS_AGENT_LABEL',defaultValue:'oc49 || oc48 || oc47')
        text(name: 'ENV_VARS', defaultValue: '', description:'''<p>
               Enter list of additional (optional) Env Vars you'd want to pass to the script, one pair on each line. <br>
               e.g.<br>
               SOMEVAR1='env-test'<br>
               SOMEVAR2='env2-test'<br>
               ...<br>
               SOMEVARn='envn-test'<br>
               </p>'''
        )
    }

  stages {
    stage('Sequential'){
      agent { label params['JENKINS_AGENT_LABEL'] }
      stages{
        stage('Build Cluster'){
          steps {
            script {
              buildno = ""
              if(params.BUILD_NUMBER == "") {
                install = build job:"ocp-common/Flexy-install", parameters: [string(name: 'INSTANCE_NAME_PREFIX', value: "${params.INSTANCE_NAME_PREFIX}"),
                  string(name: 'VARIABLES_LOCATION', value: "${params.VARIABLES_LOCATION}"),
                  text(name: 'LAUNCHER_VARS', value: "installer_payload_image: ${params.INSTALLER_PAYLOAD_IMAGE}"),
                  text(name: 'BUSHSLICER_CONFIG', value: '''services:
  AWS-CI:
    config_opts:
      region: us-east-2'''),
                  string(name: 'JENKINS_AGENT_LABEL', value: "${params.JENKINS_AGENT_LABEL}")
                ]
                buildno = install.number.toString()
              } else {
                buildno = params.BUILD_NUMBER
              }
            }
          }
        }
        stage('Prep Cluster'){
          steps {
            script {
              build job: 'scale-ci/ematysek-e2e-benchmark/logging-prep-cluster', parameters: [string(name: 'BUILD_NUMBER', value: "${buildno}"),
                booleanParam(name: 'SCALE_MACHINESETS', value: "${params.SCALE_MACHINESETS}"),
                string(name: 'ELS_INSTANCE_TYPE', value: "${params.ELS_INSTANCE_TYPE}"),
                string(name: 'FLUENTD_INSTANCE_TYPE', value: "${params.FLUENTD_INSTANCE_TYPE}"),
                string(name: 'NUM_FLUENTD_NODES', value: "${params.NUM_FLUENTD_NODES}"),
                booleanParam(name: 'DEPLOY_LOGGING', value: "${params.DEPLOY_LOGGING}"),
                string(name: 'CLO_BRANCH', value: "${params.CLO_BRANCH}"),
                booleanParam(name: 'CREATE_CLO_INSTANCE', value: "${params.CREATE_CLO_INSTANCE}"),
                string(name: 'JENKINS_AGENT_LABEL', value: "${params.JENKINS_AGENT_LABEL}")
              ]
            }
          }
        }
      }
    }
  }
}