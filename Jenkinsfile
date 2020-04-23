@Library('gitlab-mylibs@master') _
// cancel Previous running Builds to save time
cancelPreviousBuilds(env.JOB_NAME, env.BUILD_NUMBER)

pipeline{
    agent none
    triggers {       
        cron(env.BRANCH_NAME == 'master' ? '10 20 * * *' : '') // daily at 20:10 pm  --- QA  
    }
    environment {
        harboraccount = credentials('harboraccount')
        harborpasswd = credentials('harborpasswd')

    }
    options {
        //skipDefaultCheckout(true)
        //Specifying a global execution timeout of one hour, after which Jenkins will abort the Pipeline
        timeout(time: 100, unit: 'MINUTES')
        parallelsAlwaysFailFast()
        ansiColor('xterm')
    }
    parameters {
        string(name: 'WORKSPACE', defaultValue: '/app', description: 'Source code workspace')  
    }
    stages{
        stage('Core publish'){
            agent {
                kubernetes {
                defaultContainer 'jnlp'
                yamlFile 'KubernetesPod.yaml'
                }
            }
            when { not { buildingTag() } }
            steps{
                container('build'){
                    script {
                        if (env.BRANCH_NAME == 'master'){
                            echo "skip on master branch"
                        } else {
                            echo "nothing on feature branches"
                        } 
                    }
                }
            }
        }
        stage('Build & Package'){
            parallel {
                stage('Build'){
                    agent {
                        kubernetes {
                        defaultContainer 'jnlp'
                        yamlFile 'KubernetesPod.yaml'
                        }
                    }
                    stages{
                        stage('MySQL Deploy & Update'){
                            when { not { buildingTag() } }
                            options { retry(4) }
                            steps{
                                container('deploy'){
                                    script { 
                                        if (env.BRANCH_NAME == 'master'){
                                            sh """
                                            echo master branch
                                            """
                                        } else {
                                            sh """
                                            echo feature branch
                                            """
                                        }
                                    }
                                }
                                
                            }
                        }
                        stage('Tags Build, Package, Image Creation'){
                            when {  buildingTag() }
                            steps{
                                container('build'){                 
                                    // Build, Package and Image Creation                                              
                                    sh(label: 'Create Tags Image', script: """
                                    echo tag image creation
                                    """)
                                }
                                
                            }
                        }
                        stage('Build, Package, Image Creation'){
                            when { not { buildingTag() } }
                            steps{
                                container('build'){                             
                                    script {                           
                                        if (env.BRANCH_NAME == 'master'){
                                            sh """
                                            echo master 
                                            """
                                        } else {
                                            sh(label: 'Create Dev Image', script: """
                                            echo ${BRANCH_NAME} | tr 'A-Z' 'a-z' | sed s/_/-/g | sed s/[.]/-/g 
                                            echo harboraccount=${harboraccount} harborpasswd=${harborpasswd}
                                            """)
                                        }
                                    }
                                }
                                
                            }
                        }
                        stage('Deployment & E2E'){
                            when { not { buildingTag() } }
                            steps{
                                container('deploy'){
                                    script {
                                        if (env.BRANCH_NAME == 'master'){
                                            sh """
                                            echo master
                                            """
                                        } else {
                                        sh """
                                        echo feature branch
                                        """
                                        }                      
                                    }
                                }
                                container('gitversion'){
                                    script {
                                        if (env.BRANCH_NAME == 'master'){
                                            sh """
                                            echo -e "\033[32m No git version check for master branch... \033[0m"
                                            """
                                        } else {
                                            sh """
                                            echo -e "\033[32m start verifying Dev deployment git version... \033[0m"                 
                                            """
                                        } 
                                    }
                                }
                                container('e2etest'){
                                    script {
                                        if (env.BRANCH_NAME == 'master'){
                                            sh """                           
                                            echo -e "\033[32m No E2E Tests for QA environment \033[0m"
                                            """
                                        } else {
                                            sh(label: 'UI Test Part1', script: """
                                                echo Part1
                                            """)
                                            sh(label: 'UI Test Part2', script: """
                                                echo Part2
                                            """)
                                        }   
                                    }                
                                }
                            }
                        }
                    }
                }
                stage('Smoke'){
                    agent {
                        kubernetes {
                        defaultContainer 'jnlp'
                        yamlFile 'KubernetesPod-Smoke.yaml'
                        }
                    }
                    when { not { buildingTag() } }
                    stages{
                        stage('Smoke Test'){
                            steps{
                                echo "smoke test"
                                container('build'){
                                    sh(label: 'Smoke Test', script: """
                                        echo Smoke Test
                                    """)
                                }
                            }  
                        }
                        stage('Python Unit Test'){
                            steps{
                                echo "python test"
                                container('build'){
                                    /* Seperate test steps so easy to analyse test time distribution - Jason */
                                    sh(label: 'Unit Test', script: """
                                        echo python test
                                    """)                                  
                                }
                                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: true, reportDir: 'reports/python_coverage', reportFiles: 'index.html', reportName: 'python coverage Report', reportTitles: 'python coverage'])
                                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: true, reportDir: 'reports/python_test_reports', reportFiles: 'index.html', reportName: 'python test Report', reportTitles: 'python test'])
                            }
                        }
                         
                    }
                }
            }  
        }    
        
        
    }
    
    post {
        always {           
            echo 'One way or another, I have finished'
        }
        failure {
            echo 'I failed'                        
        }
        success {
            echo 'SUCCESS'                                           
        }
    } 
}
