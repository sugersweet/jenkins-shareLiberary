## jenkins-shareLiberary共享库

<table><tr><td bgcolor="orange">为什么要使用Jenkins共享库？</td></tr>
<tr><td bgcolor="orange">如何使用该Jenkins共享库？</td></tr>
<tr><td bgcolor="orange">该项目共享库包括哪些功能？</td></tr></table>

##### 为什么要使用Jenkins共享库？
###### 随着团队项目的增多，需要向各项目提供CI/CD服务，且各个项目对于CI/CD服务的要求重合度非常高，为每个项目维护Jenkins任务或者Jenkinsfile费时费力且不便于CI/CD任务更新升级。因此，使用统一的流水线模版，将重复和重合度高的模块提取出来集合到Jenkins共享库，方便各个项目调用，更新升级也更为方便和解耦合。

##### 如何使用该Jenkins共享库？

###### 1. 在Jenkins控制页面中选择"Manage Jenkins" --> "Configure System" --> "Global Pipeline Libraries" --> 填入"Name"别名, "Default version"填入该项目已存在的分支名称如master, "Source Code Management"选择Git并填入该项目的github路径，具体如下图所示：


###### 2. In Jenkinsfile use @Library('gitlab-mylibs@master') _ to include this Name alias in Jenkins Configure System.
