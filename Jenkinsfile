pipeline {
  agent {
    dockerfile { filename 'Dockerfile.build' }
  }

  environment {
    // Jenkins doesnt set a $HOME and causes dotnet to fail
    // https://issues.jenkins-ci.org/browse/JENKINS-31225
    HOME="${WORKSPACE}"
    output_dir="${WORKSPACE}/output/build"
    artifactory_url=getConfig('artifactory_url')
    Version=getVersion()
    Location="${WORKSPACE}/Amazon.ElastiCacheCluster/Amazon.ElastiCacheCluster.csproj"
  }

  stages {
    stage('Setup') {
      steps {
        script {
          dir("${WORKSPACE}/output") {
            deleteDir()
          }
          setDisplayInfo(Version)
        }
      }
    }
    stage('Pack') {
      steps {
        sh "dotnet pack ${Location} -c:Release -o ${output_dir} /p:Version=${Version} /p:PackageVersion=${Version}"
      }
    }
    stage('Publish') {
      when{
        branch 'master'
      }
      steps {
        withCredentials([usernameColonPassword(credentialsId: 'artifactory:deployer', variable: 'KEY')]) {
          sh '''
          set +x
          dotnet nuget push ${output_dir}/*.nupkg -s ${artifactory_url} -k ${KEY}
          '''
        }
      }
    }
  }
  post{
    always {
      dir("${WORKSPACE}/output") {
        deleteDir()
      }
      setDisplayInfo(Version)
    }
  }
}