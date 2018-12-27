properties properties: [
        [$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '30', numToKeepStr: '10']],
        [$class: 'GithubProjectProperty', displayName: '', projectUrlStr: 'https://github.com/hypery2k/Jenkins2Go']
]

timeout(60) {
  node('mac') {
      // Jenkins makes these variables available for each job it runs
      def packageName = "com.jenkins2go"
      def buildNumber = env.BUILD_NUMBER
      def workspace = env.WORKSPACE
      def appHome = 'app'
      def buildUrl = env.BUILD_URL
      def appName = 'Jenkins2Go'
      def scheme = 'Jenkins2Go'
      def buildSDK = 'iphoneos12.1' // see `xcodebuild -showsdks`
      def provisioningProfile = 'Distribution - Jenkins2Go'

      // PRINT ENVIRONMENT TO JOB
      echo "workspace directory is $workspace"
      echo "build URL is $buildUrl"
      echo "build Number is $buildNumber"

      try {
          stage('Checkout') {
              checkout scm
              sh 'git submodule init && git submodule update'
              sh 'rm -rf target/*'
          }

          stage('Build') {
              sh "agvtool new-version -all ${buildNumber}"
              sh "xcodebuild clean build -sdk ${buildSDK} PROVISIONING_PROFILE=\"${provisioningProfile}\" -configuration Release"
          }

          stage('Test') {
          }

          stage('Package') {
              sh "mkdir -p target"
              sh "xcodebuild clean build -scheme \"${scheme}\" -sdk \"${buildSDK}\" -archivePath \"target/${scheme}.xcarchive\" -configuration Release PROVISIONING_PROFILE=\"${provisioningProfile}\" archive "
              sh "xcodebuild -exportArchive -sdk \"${buildSDK}\" -archivePath target/${scheme}.xcarchive/  -exportPath \"./target/${scheme}_${buildNumber}.ipa\""
          }

          stage('Upload App') {
              sh "fastlane pilot upload --ipa target/${scheme}_${buildNumber}.ipa"
          }

      } catch (e) {
          mail subject: "${env.JOB_NAME} (${env.BUILD_NUMBER}): Error on build", to: 'jenkins2go-app@martinreinhardt-online.de', body: "Please go to ${env.BUILD_URL}."
          throw e
      }
  }
}
