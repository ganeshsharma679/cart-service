pipeline {
  stages {
  stage('Build') {
     steps {
    git url: "https://github.com/siamaksade/cart-service.git"
    sh "mvn package"
    stash name:"jar", includes:"target/cart.jar"
     }
  }
  stage('Test') {
     steps {
    parallel(
      "Cart Tests": {
        sh "mvn verify -P cart-tests"
      },
      "Discount Tests": {
        sh "mvn verify -P discount-tests"
      }
      }
    )
  }
  stage('Build Image') {
     steps {
    unstash name:"jar"
    sh "oc start-build cart --from-file=target/cart.jar --follow"
     }
  }
  stage('Deploy') {
    steps {
        script {
            openshift.withCluster() {
                openshift.withProject() {
                  def rm = openshift.selector("dc", templateName).rollout().latest()
                  timeout(5) { 
                    openshift.selector("dc", templateName).related('pods').untilEach(1) {
                      return (it.object().status.phase == "Running")
                    }
                  }
                }
            }
        }
      }
  }
  stage('System Test') {
    sh "curl -s -X POST http://cart:8080/api/cart/dummy/666/1"
    sh "curl -s http://cart:8080/api/cart/dummy | grep 'Dummy Product'"
  }
}
}
