## jenkins-shareLiberary

####How to use jenkins share lieraries in Jenkins or Jenkinsfile
###### 1. In Jenkins "Manage Jenkins" --> "Configure System" --> "Global Pipeline Libraries" --> add "Name" alias, "Default version" branch name, "Source Code Management" use Git with github path to this project.
###### 2. In Jenkinsfile use @Library('gitlab-mylibs@master') _ to include this Name alias in Jenkins Configure System.
