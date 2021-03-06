#!/usr/bin/groovy
def localItestPattern = ""
try {
  localItestPattern = ITEST_PATTERN
} catch (Throwable e) {
  localItestPattern = "*KT"
}

def localFailIfNoTests = ""
try {
  localFailIfNoTests = ITEST_FAIL_IF_NO_TEST
} catch (Throwable e) {
  localFailIfNoTests = "false"
}

def versionPrefix = ""
try {
  versionPrefix = VERSION_PREFIX
} catch (Throwable e) {
  versionPrefix = "1"
}

def canaryVersion = "${versionPrefix}.${env.BUILD_NUMBER}"

def fabric8Console = "${env.FABRIC8_CONSOLE ?: ''}"

node ('kubernetes'){
  git 'https://github.com/flxvh/jenkins-test.git'

  kubernetes.pod('buildpod').withImage('fabric8/maven-builder')
      .withPrivileged(true)
      .withHostPathMount('/var/run/docker.sock','/var/run/docker.sock')
      .withEnvVar('DOCKER_CONFIG','/home/jenkins/.docker/')
      .withSecret('jenkins-docker-cfg','/home/jenkins/.docker')
      .withSecret('jenkins-maven-settings','/root/.m2')
      .withServiceAccount('jenkins')
      .inside {

    mavenCanaryRelease{
      version = canaryVersion
    }

    // TODO docker push?

    mavenIntegrationTest{
      environment = 'Testing'
      failIfNoTests = localFailIfNoTests
      itestPattern = localItestPattern
    }

    mavenRollingUpgrade{
      environment = 'Staging'
      stageDomain = STAGE_DOMAIN
    }

    approve{
      room = null
      version = canaryVersion
      console = fabric8Console
      environment = 'Staging'
    }

    mavenRollingUpgrade{
      environment = 'Production'
      stageDomain = PROMOTE_DOMAIN
    }
  }
}
