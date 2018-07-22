node {
  def namespace = 'devops'
  def appName = 'gerua'
  def feSvcName = "${appName}-frontend"
  def DockerRegistry = sh(returnStdout: true, script: "cat ../docker_resgistry | cut -f2 -d'='").trim()
  def imageTag = "${DockerRegistry}:${namespace}-${appName}-${env.BRANCH_NAME}-${env.BUILD_NUMBER}"

  checkout scm

  stage 'Build image'
  sh("docker build -t ${imageTag} .")

  stage 'Run Go tests'
  sh("docker run ${imageTag} go test")

  stage 'Push image to registry'
  sh("docker push ${imageTag}")

  stage "Deploy Application"
  switch (env.BRANCH_NAME) {
    // Roll out to canary environment
    case "canary":
        // Change deployed image in canary to the one we just built
        sh("sed -i.bak 's#docker-registry/gerua:1.0.0#${imageTag}#' ./k8s/canary/*.yaml")
        sh("kubectl --namespace=${namespace} apply -f k8s/services/")
        sh("kubectl --namespace=${namespace} apply -f k8s/canary/")
        sh("echo http://`kubectl --namespace=${namespace} get service/${feSvcName} --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > ${feSvcName}")
        break

    // Roll out to production
    case "master":
        // Change deployed image in canary to the one we just built
        sh("sed -i.bak 's#docker-registry/gerua:1.0.0#${imageTag}#' ./k8s/production/*.yaml")
        sh("kubectl --namespace=${namespace} apply -f k8s/services/")
        sh("kubectl --namespace=${namespace} apply -f k8s/production/")
        sh("echo http://`kubectl --namespace=${namespace} get service/${feSvcName} --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > ${feSvcName}")
        break
  }
}
