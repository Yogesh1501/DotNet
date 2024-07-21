pipeline { 
    agent any 
    tools { 
        jdk 'jdk17' 
    } 
    environment { 
        SCANNER_HOME = tool 'sonar-scanner' 
    } 
    stages { 
        stage('Clean Workspace') { 
            steps { 
                echo 'Cleaning workspace...'
                cleanWs() 
            } 
        } 
        stage('Checkout From Git') { 
            steps { 
                echo 'Checking out from Git...'
                git branch: 'main', url: 'https://github.com/Yogesh1501/DotNet.git' 
            } 
        } 
        stage("Sonarqube Analysis") { 
            steps { 
                echo 'Starting SonarQube analysis...'
                withSonarQubeEnv('sonar-server') { 
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Dotnet-Webapp \
                        -Dsonar.projectKey=Dotnet-Webapp
                    ''' 
                } 
            } 
        } 
        stage("Quality Gate") { 
            steps { 
                echo 'Waiting for Quality Gate...'
                script { 
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                } 
            } 
        } 
        stage("TRIVY File Scan") { 
            steps { 
                echo 'Starting TRIVY file scan...'
                sh "trivy fs . > trivy-fs_report.txt" 
            } 
        } 
        stage("OWASP Dependency Check") { 
            steps { 
                echo 'Starting OWASP Dependency Check...'
                dependencyCheck additionalArguments: '--scan ./ --format XML', odcInstallation: 'DP-Check' 
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml' 
            } 
        } 
        stage("Docker Build & Tag") { 
            steps { 
                echo 'Building and tagging Docker image...'
                script { 
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {    
                        sh "make image" 
                    } 
                } 
            } 
        } 
        stage("TRIVY Image Scan") { 
            steps { 
                echo 'Starting TRIVY image scan...'
                sh "trivy image yogesh1501/dotnet-monitoring:latest > trivy.txt"  
            } 
        } 
        stage("Docker Push") { 
            steps { 
                echo 'Pushing Docker image...'
                script { 
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {    
                        sh "make push" 
                    } 
                } 
            } 
        }
        stage("Deploy to container"){ 
            steps{ 
                sh "docker run -itd --name dotnet -p 5000:5000 yogesh1501/dotnet-monitoring:latest" 
            }  
        }
    } 
}
