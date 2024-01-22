pipeline {
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'nodejs16'
    }
    environment{
        SCANNER_HOME=tool 'sonar-scanner'
        Version="${env.BUILD_ID}"
        GithubUser="${params.GithubUser}"
        GithubRepo="${params.GithubRepo}"
        DockerUser="${params.DockerUser}"
        DockerRepo="${params.DockerRepo}"
        MyName="${params.MyName}"
        MyEmail="${params.MyEmail}"
    }
    stages {
        stage('Sonar Scan'){
            steps{
                script{
                    withSonarQubeEnv('sonar-server') {
                        sh "${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=${DockerRepo}"
                    }
                }
            }
        }
        stage('Quality Gate'){
            steps{
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
            }
        }
        stage('NPM Installation'){
            steps{
                sh 'npm install'
            }
        }
        stage('Trivy FS Scan'){
            steps{
                sh 'trivy fs . > trivyfs.txt'
            }
        }
        stage('OWASP Scan'){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Docker Image'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker') {
                    sh '''
                        docker build -t $DockerRepo:$Version .
                        docker tag $DockerRepo:$Version $DockerUser/$DockerRepo:$Version
                        docker tag $DockerRepo:$Version $DockerUser/$DockerRepo:latest
                        docker push $DockerUser/$DockerRepo:$Version
                        docker push $DockerUser/$DockerRepo:latest
                    '''
                    }
                }
            }
        }
        stage('Trivy Image Scan'){
            steps{
                sh 'trivy image $DockerRepo:$Version > trivyimage.txt'
            }
        }
        stage('Manifest Pull'){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/jeetu844/Shopping-reactJS-manifest.git'
            }
        }
        stage('Update Manifest'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'github-token', variable: 'gitcred')]) {
                        sh '''
                            sed -i "s|image: .*|image: ${DockerUser}/$DockerRepo:${Version}|g" deployment.yml
                            git config --global user.email ${MyEmail}
                            git config --global user.name ${MyName}
                            git add deployment.yml
                            git commit -a -m "Update Manifest with $DockerUser/$DockerRepo:$Version"
                            git push https://$gitcred@github.com/$GithubUser/$GithubRepo HEAD:main
                        '''
                    }
                }
            }
        }
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
    }
}
