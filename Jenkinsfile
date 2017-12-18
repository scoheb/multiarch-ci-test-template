properties(
  [
    parameters(
      [
        string(
          defaultValue: 'x86_64',
          description: 'A comma separated list of architectures to run the test on. Valid values include [x86_64, ppc64le, aarch64, s390x].',
          name: 'ARCHES'
        )
      ]
    )
  ]
)

library identifier: "multiarch-ci-libraries@scoheb-dev",
        retriever: modernSCM([$class: 'GitSCMSource',
                              remote: "https://github.com/scoheb/multiarch-ci-libraries",
                              traits: [[$class: 'jenkins.plugins.git.traits.BranchDiscoveryTrait'],
                                       [$class: 'RefSpecsSCMSourceTrait',
                                        templates: [[value: '+refs/heads/*:refs/remotes/@{remote}/*'],
                                                    [value: '+refs/pull/*:refs/remotes/origin/pr/*']]]]])


import com.redhat.multiarch.ci.Slave

def List arches = params.ARCHES.tokenize(',')
def Boolean runOnProvisionedHosts = false
def Boolean installAnsible = true
def tenant = "continuous-infra"
def dockerUrl = "172.30.1.1:5000"
def krbPrincipal = "jenkins/ci-ops-jenkins.rhev-ci-vms.eng.rdu2.redhat.com@REDHAT.COM"
def keytab = "KEYTAB"

parallelMultiArchTest(
  arches,
  tenant,
  dockerUrl,
  krbPrincipal,
  keytab,
  runOnProvisionedHosts,
  installAnsible,
  { Slave slave ->
    /******************************************************************/
    /* TEST BODY                                                      */
    /* @param provisionedSlave    Name of the provisioned host.       */
    /* @param arch                Architeture of the provisioned host */
    /******************************************************************/
    stage ('Download Test Files') {
      checkout scm
    }

    // TODO insert test body here
    stage ('Run Test') {
      sh 'ansible-playbook tests/ansible-playbooks/*/playbook.yml'
    }

    stage ('Archive Test Output') {
      archiveArtifacts artifacts: 'tests/ansible-playbooks/**/artifacts/*', fingerprint: true
      try {
        junit 'tests/ansible-playbooks/**/reports/*.xml'
      } catch (e) {
        // We don't care if this step fails
      }
    }

    /*****************************************************************/
    /* END TEST BODY                                                 */
    /* Do not edit beyond this point                                 */
    /*****************************************************************/
  },
  { exception, arch ->
    println("Exception ${exception} occured on ${arch}")
    if (arch.equals("x86_64") || arch.equals("ppc64le")) {
      currentBuild.result = 'FAILURE'
    }
  }
)
