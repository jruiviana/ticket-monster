node("master") {

   /*
   * Clone/Checkout project from SCM server
   */
  //git([url: 'ssh://git@ciserver.unisys.com.br/home/git/ticket_monster.git', branch: 'master'])
  checkout scm

  /*
   * Check and configure Maven Install - required to determine the correct maven version
   */
  def mvnHome = tool 'Maven3'

  /*
   * Build project using Maven
   */
  stage "Build"
  sh '${mvnHome}/bin/mvn clean package -Ppostgresql'
  step([$class: 'ArtifactArchiver', artifacts: '**/target/*.war', fingerprint: true])

  /*
   * Run integration tests using Arquillian. Tests run remote on cdserver.
   */
  stage "Integration Test"
  sh '${mvnHome}/bin/mvn test -Parq-wildfly-remote'
  step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])

  /*
   * Run sonarqube analysis
   */
  stage "Sonar Analisys"
  sh '${mvnHome}/bin/mvn verify sonar:sonar -P default,sonar'

  /*
   * Last stage - Publish to staging environment.
   */
  stage "Publish For Wildfly Staging"
  sh '${mvnHome}/bin/mvn wildfly:deploy'

}