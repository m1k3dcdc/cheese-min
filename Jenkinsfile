def templatePath = './cheese-min-template.yaml'
def templateName = 'cheese-min-template'
def APPName = 'cheese-min'
pipeline {
  agent any

  options {
    timeout(time: 10, unit: 'MINUTES') 
  }
  
  stages {
    stage('preamble') {
        steps {
            script {
                echo "STAGE: preamble"
                openshift.withCluster() {
                    openshift.withProject() {
                        echo "*** Using project: ${openshift.project()}"           
                    }
                }
            }
        }
    }
    stage('cleanup') {
      steps {
        script {
            echo "STAGE: cleanup"
            openshift.withCluster() {
                openshift.withProject() {
                  //openshift.selector("all", [ template : templateName ]).delete() 
                  //openshift.selector( 'dc', [ environment:'qe' ] ).delete()

                  if (openshift.selector("is", APPName).exists()) { 
                    openshift.selector("is", APPName).delete()
                    echo "*** is delete"
                  }
                  if (openshift.selector("bc", APPName).exists()) { 
                    openshift.selector("bc", APPName).delete()
                    echo "*** bc delete"
                  }
                  if (openshift.selector("dc", APPName).exists()) { 
                    openshift.selector("dc", APPName).delete()
                    echo "*** dc delete"
                  }
                  if (openshift.selector("deploy", APPName).exists()) { 
                    openshift.selector("deploy", APPName).delete()
                    echo "*** deploy delete"
                  }                  
                  if (openshift.selector("svc", APPName).exists()) { 
                    openshift.selector("svc", APPName).delete()
                    echo "*** svc delete"
                  }
                  if (openshift.selector("route", APPName).exists()) { 
                    openshift.selector("route", APPName).delete()
                    echo "*** route delete"
                  }
                }
            }
        }
      }
    }
    stage('create') {
      steps {
        script {
            echo "STAGE: create"
            openshift.withCluster() {
                openshift.withProject() {
                  //openshift.newApp(templatePath) 
                  
                    def templateSelector = openshift.selector("template", templateName)                  
                    def templateExists = templateSelector.exists()
                    def template
                    if (templateExists) {
                        templateSelector.describe()
                        openshift.selector("template", templateName).delete()
                        echo "*** template delete"
                     } 
                      echo "Create Template"
                      sh "oc process -f ${templatePath} | oc create -f -"
                      //template = openshift.create(templatePath, "-f").object()
                      //template = templateSelector.object()
                      //template.describe()               
                }
            }
        }
      }
    }
    stage('build') {
      steps {
        script {
            echo "STAGE: build"
            openshift.withCluster() {
                openshift.withProject() {
                  echo "*** Start Build"

                  if (!openshift.selector("bc", APPName).exists()) {
                        openshift.newBuild("--name=${APPName}", "--image=docker.io/m1k3pjem/cheese-java-pipeline", "--binary")
                  }    
                  def startBuildLog = openshift.selector("bc", APPName).startBuild("--from-dir=.", "--follow")
                  startBuildLog.logs('-f')
                  //sh "oc start-build ${APPName} --from-dir=."                                 
                  
                  def builds = openshift.selector("bc", APPName).related('builds')
                  echo "*** BUILS related"
                  timeout(5) { 
                    builds.untilEach(1) {
                      return (it.object().status.phase == "Complete")
                    }
                  }
                }
            }
        }
      }
    } 
    stage('deploy') {
      steps {
        script {
            echo "STAGE: deploy"
            openshift.withCluster() {
                openshift.withProject() {
                  //openshift.selector("dc", APPName).rollout()
                  echo "*** Deploy Config"
                    
                  if(!openshift.selector("dc", APPName).exists()){
                    //openshift.newApp('hello-java-spring-boot', "--as-deployment-config").narrow('svc')
                    sh "oc rollout latest dc/${APPName}"
                  }

                  if (!openshift.selector("route", APPName).exists()) {
                    echo "### Route " + APPName + " does not exist, exposing service ..." 
                    def service = openshift.selector("service", APPName)
                    service.expose()
                  }

                  if (openshift.selector("bc", APPName).exists()) {
                    echo "### BC " + APPName + " exist, create Trigger ..." 
                    sh "oc set triggers bc/${APPName} --from-github=false"
                  }
/*
                  def deployPod = openshift.selector("dc", APPName)
                  deployPod.logs("-f")
                  echo "*** Pod related"
                  deployPod.related("pods").untilEach {
                    return it.object().status.phase == 'Running'
                  }
  */                
/*                 
                  timeout(5) { 
                    deployPod.untilEach(1) {
                      return (it.object().status.phase == "Running")
                    }
                  }  */
                  
                  timeout(5) { 
                    echo "*** Timeout related"
                    openshift.selector("dc", APPName).related('pods').untilEach(1) {
                      return (it.object().status.phase == "Running")
                    }
                  }  
                  
                }
            }
        }
      }
    } 
  /*  stage('tag') {
      steps {
        script {
            echo "STAGE: tag"
            openshift.withCluster() {
                openshift.withProject() {
                  openshift.tag("${APPName}:latest", "${APPName}-staging:latest") 
                }
            }
        }
      }
    } */
  }
}
