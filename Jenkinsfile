pipeline {
    environment {
      branchname =  env.BRANCH_NAME.toLowerCase()
      kubeconfig = getKubeconf(env.branchname)
      registryCredential = 'jenkins_registry'
    }
  
    agent {
      node { label 'AGENT-NODES' }
    }

    options {
      buildDiscarder(logRotator(numToKeepStr: '5', artifactNumToKeepStr: '5'))
      disableConcurrentBuilds()
      skipDefaultCheckout()
    }
  
    stages {

        stage('CheckOut') {            
            steps { checkout scm }            
        }

        stage('Build') {
          when { anyOf { branch 'master'; branch 'main';  } } 
          steps {
            script {
              imagename1 = "registry.sme.prefeitura.sp.gov.br/${env.branchname}/sigpae-docs"
              dockerImage1 = docker.build(imagename1, "-f Dockerfile .")
              docker.withRegistry( 'https://registry.sme.prefeitura.sp.gov.br', registryCredential ) {
              dockerImage1.push()
              }
              sh "docker rmi $imagename1"
            }
          }
        }

        stage('Deploy'){
            when { anyOf {  branch 'master'; branch 'main'; } }        
            steps {
                script{
                    withCredentials([file(credentialsId: "${kubeconfig}", variable: 'config')]){
                       sh 'if [ -d "$home/.kube/config" ]; then rm -f "$home/.kube/config"; fi'
                       sh('cp $config '+"$home"+'/.kube/config')
                       sh "kubectl rollout restart deployment/sigpae-docs -n sme-sigpae"						
                       sh('rm -f '+"$home"+'/.kube/config')
                    }
                }
            }           
        }    
    }
}

def getKubeconf(branchName) {
    if("main".equals(branchName)) { return "config_prd"; }
    else if ("master".equals(branchName)) { return "config_prd"; }
    else if ("homolog".equals(branchName)) { return "config_hom"; }
    else if ("release".equals(branchName)) { return "config_hom"; }
    else if ("release2".equals(branchName)) { return "config_hom"; }
    else if ("development".equals(branchName)) { return "config_dev"; }
    else if ("develop".equals(branchName)) { return "config_dev"; }
}
