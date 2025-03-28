import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
withEnv(['AZURE_SUBSCRIPTION_ID=2e4b8c87-efc1-46d3-b489-17e0cd5826d0',
        'AZURE_TENANT_ID=d6156c80-c0e0-4b62-949b-0e874da052b1']) {
stage('init') {
checkout scm
}
  
stage('build') {
sh 'mvn clean package'
}
  
stage('deploy') {
      def resourceGroup = 'jenkins-get-started-rg'
      def webAppName = 'cloud8-taotao'
      // login Azure
withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
}
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
//sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
sh 'az webapp deploy --resource-group jenkins-get-started-rg --name cloud8-taotao --src-path target/calculator-1.0.war --type war'
  // log out
sh 'az logout'
}
}
}
