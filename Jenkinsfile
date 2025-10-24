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
/*
                        // Create a Selector capable of selecting all service accounts in mycluster's default project
                        def saSelector = openshift.selector( 'serviceaccount' )                    
                        // Prints `oc describe serviceaccount` to Jenkins console
                        saSelector.describe()                    
                        // Selectors also allow you to easily iterate through all objects they currently select.
                        saSelector.withEach { // The closure body will be executed once for each selected object.
                            // The 'it' variable will be bound to a Selector which selects a single
                            // object which is the focus of the iteration.
                            echo "Service account: ${it.name()} is defined in ${openshift.project()}"
                        }                    
                        // Prints a list of current service accounts to the console
                        echo "There are ${saSelector.count()} service accounts in project ${openshift.project()}"
                        echo "They are named: ${saSelector.names()}"
*/                        
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
                  //sh "oc delete template ${templateName}"
/*                  
                  if (openshift.selector("template", templateName).exists()) { 
                    openshift.selector("template", templateName).delete()
                    echo "*** template delete"
                  }
                  sh "oc process -f ${templatePath} | oc create -f -"
*/                  
                  
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

                  def buildConfigExists = openshift.selector("bc", APPName).exists()
                    
                  echo "### BuildConfig: " + APPName + " exists, start new build ..."
                  if (!buildConfigExists) {
                        echo "### newBuild " + APPName
                        openshift.newBuild("--name=${APPName}", "--image=docker.io/m1k3pjem/hello-java-spring-boot", "--binary")
/*
                        if (!openshift.selector("route", APPName).exists()) {
                            echo "### Route " + APPName + " does not exist, exposing service ..." 
                            def service = openshift.selector("service", APPName)
                            service.expose()
                        } else {
                            echo "### Route " + APPName + " exist" 
                        }*/
                  }    
                  def startBuildLog = openshift.selector("bc", APPName).startBuild("--from-dir=.")
                  startBuildLog.logs('-f')
                                 
                  //def buildSelector = openshift.selector("bc", APPName).startBuild()
                  //buildSelector.logs('-f')
                  /*
                  def builds = openshift.selector("bc", APPName).related('builds')
                  echo "*** BUILS related"
                  timeout(5) { 
                    builds.untilEach(1) {
                      return (it.object().status.phase == "Complete")
                    }
                  }*/
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
                  
                  def deployment = openshift.selector("dc", APPName)
    
                  if(!deployment.exists()){
                    //openshift.newApp('hello-java-spring-boot', "--as-deployment-config").narrow('svc').expose()
                   // sh "oc apply dc/${APPName}"
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
                    //oc set triggers bc/APPName --from-github --webhook-secret=mysecret123
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
                  /*
                  timeout(5) { 
                    echo "*** Timeout"
                    openshift.selector("deploy", APPName).related('pods').untilEach(1) {
                      return (it.object().status.phase == "Running")
                    }
                  }  */
                  
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
