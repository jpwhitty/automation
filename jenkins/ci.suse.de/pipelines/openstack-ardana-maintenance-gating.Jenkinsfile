/**
 * The openstack-ardana-maintenance-gating Jenkins Pipeline
 */

def ardana_lib = null

pipeline {
  // skip the default checkout, because we want to use a custom path
  options {
    skipDefaultCheckout()
    timestamps()
  }

  agent {
    node {
      label 'cloud-trigger'
      customWorkspace "${JOB_NAME}-${BUILD_NUMBER}"
    }
  }

  stages {
    stage('Setup workspace') {
      steps {
        script {
          if (ardana_env == '') {
            error("Empty 'ardana_env' parameter value.")
          }
          if (maint_updates == '') {
            error("Empty 'maint_updates' parameter value.")
          }
          currentBuild.displayName = "#${BUILD_NUMBER}: ${maint_updates}"

          sh('''
            git clone $git_automation_repo --branch $git_automation_branch automation-git
          ''')

          ardana_lib = load "$WORKSPACE/automation-git/jenkins/ci.suse.de/pipelines/openstack-ardana.groovy"
        }
      }
    }

    stage('Trigger jobs') {
      // Do not abort all stages if one of them fails
      failFast false
      parallel {
        stage('Run cloud deploy job') {
          when {
            expression { deploy == 'true' }
          }
          steps {
            script {
              // reserve a resource here for the openstack-ardana job, to avoid
              // keeping a cloud-ardana-ci worker busy while waiting for a
              // resource to become available.
              ardana_lib.run_with_reserved_env(reserve_env == 'true', ardana_env, "${ardana_env}-deploy") {
                reserved_env ->
                def slaveJob = ardana_lib.trigger_build("cloud-ardana${version}-job-entry-scale-kvm-maintenance-update-x86_64", [
                  string(name: 'ardana_env', value: reserved_env),
                  string(name: 'reserve_env', value: "false"),
                  string(name: 'cloudsource', value: "$cloudsource"),
                  string(name: 'maint_updates', value: "$maint_updates"),
                  string(name: 'update_after_deploy', value: "false"),
                  string(name: 'rc_notify', value: "true"),
                  string(name: 'cleanup', value: "on success"),
                  string(name: 'git_automation_repo', value: "$git_automation_repo"),
                  string(name: 'git_automation_branch', value: "$git_automation_branch"),
                  string(name: 'os_cloud', value: "engcloud-cloud-ci-private")
                ], false)
              }
            }
          }
        }

        stage('Run cloud update job') {
          when {
            expression { deploy_and_update == 'true' }
          }
          steps {
            script {
              // reserve a resource here for the openstack-ardana job, to avoid
              // keeping a cloud-ardana-ci worker busy while waiting for a
              // resource to become available.
              ardana_lib.run_with_reserved_env(reserve_env == 'true', ardana_env, "${ardana_env}-update") {
                reserved_env ->
                def slaveJob = ardana_lib.trigger_build("cloud-ardana${version}-job-entry-scale-kvm-maintenance-update-x86_64", [
                  string(name: 'ardana_env', value: reserved_env),
                  string(name: 'reserve_env', value: "false"),
                  string(name: 'cloudsource', value: "$cloudsource"),
                  string(name: 'maint_updates', value: "$maint_updates"),
                  string(name: 'update_after_deploy', value: "true"),
                  string(name: 'rc_notify', value: "true"),
                  string(name: 'cleanup', value: "on success"),
                  string(name: 'git_automation_repo', value: "$git_automation_repo"),
                  string(name: 'git_automation_branch', value: "$git_automation_branch"),
                  string(name: 'os_cloud', value: "engcloud-cloud-ci-private")
                ], false)
              }
            }
          }
        }
      }
    }
  }
  post{
    cleanup {
      cleanWs()
    }
  }
}
