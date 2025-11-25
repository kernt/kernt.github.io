Goto -> Jenkins -> Manage Jenkins -> Manage Credentials -> Stored scoped to jenkins -> global -> Add Credentials [ GIVE YOUR DOCKER HUB CREDENTIALS ]

Next, we will create GitHub credentials

The Global Credentials Dashboard should look like this

The Global Credentials Dashboard should look like this

![](https://miro.medium.com/v2/resize:fit:700/1*ztOWl7_E1JT7wfOQVlUd2A.png)

**Step 11 — Add Maven in Global Tool Configuration**

![](https://miro.medium.com/v2/resize:fit:700/1*t1Y69bll_HVO47hycqWlow.png)

**Step 12 — Add Jenkins Shared Library**

Go to Manage Jenkins → Configure System → Global Pipeline Libraries →

![](https://miro.medium.com/v2/resize:fit:700/1*FrsU3iNU2roddo4q-_FRKA.png)

![](https://miro.medium.com/v2/resize:fit:700/1*n1jghCLgbnEeAJdO_4zvvw.png)

**Library name** — jenkins-shared-library

**Default Version** — main

**Project Repository** — [https://github.com/writetoritika/jenkins-shared-library1.git](https://github.com/DEVOPSWITH-WEB-DEV/jenkins-shared-library1.git)

Click on Apply and Save

**Step 13 — Build, Deploy and Test Jenkins Pipeline**

Create new Pipeline: Go to Jenkins Dashboard

click on New Item Pipeline Name: EKS-Demo-Pipeline

select Pipeline Add pipeline script

Add the below script

```
@Library('jenkins-shared-library@main') _  
pipeline {  
  
  agent any  
    
  parameters {  
 choice(name: 'action', choices: 'create\nrollback', description: 'Create/rollback of the deployment')  
    string(name: 'ImageName', description: "Name of the docker build", defaultValue: "kubernetes-configmap-reload")  
 string(name: 'ImageTag', description: "Name of the docker build",defaultValue: "v1")  
 string(name: 'AppName', description: "Name of the Application",defaultValue: "kubernetes-configmap-reload")  
    string(name: 'docker_repo', description: "Name of docker repository",defaultValue: "writetoritika")  
  }  
        
  tools{   
        maven 'maven3'  
    }  
    stages {  
        stage('Git-Checkout') {  
            when {  
    expression { params.action == 'create' }  
   }  
            steps {  
                gitCheckout(  
                    branch: "main",  
                    url: "https://github.com/writetoritika/spring-cloud-kubernetes.git"  
                )  
            }  
        }  
        stage('Build-Maven'){  
            when {  
    expression { params.action == 'create' }  
   }  
      steps {  
          dir("${params.AppName}") {  
           sh 'mvn clean package'  
          }  
      }  
     }  
     stage("DockerBuild and Push") {  
         when {  
    expression { params.action == 'create' }  
   }  
         steps {  
             dir("${params.AppName}") {  
                 dockerBuild ( "${params.ImageName}", "${params.docker_repo}" )  
             }  
         }  
     }  
     stage("Docker-CleanUP") {  
         when {  
    expression { params.action == 'create' }  
   }  
         steps {  
             dockerCleanup ( "${params.ImageName}", "${params.docker_repo}" )  
   }  
  }  
       stage("Ansible Setup") {  
         when {  
    expression { params.action == 'create' }  
   }  
         steps {  
             sh 'ansible-playbook ${WORKSPACE}/kubernetes-configmap-reload/server_setup.yml'  
   }  
  }  
     stage("Create deployment") {  
   when {  
    expression { params.action == 'create' }  
   }  
         steps {  
             sh 'echo ${WORKSPACE}'  
             sh 'kubectl create -f ${WORKSPACE}/kubernetes-configmap-reload/kubernetes-configmap.yml'  
         }  
     }  
     stage ("wait_for_pods"){  
     steps{  
                
                sh 'sleep 300'  
               
     }  
     }  
  stage("rollback deployment") {  
         steps {                                     
               sh """  
                    kubectl delete deploy ${params.AppName}  
         kubectl delete svc ${params.AppName}  
      """          
         }  
     }  
    }  
}
```

