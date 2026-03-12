pipeline {
    agent any

    environment {
        REGISTRY = "xxxxxxxx.dkr.ecr.ap-south-1.amazonaws.com"
        IMAGE = "hotel-booking-service"
        SONARQUBE = "sonarqube"           // Jenkins SonarQube installation
        VAULT_ADDR = "https://vault.example.com"
    }

    options {
        timestamps()
        timeout(time: 90, unit: 'MINUTES')
    }

    stages {

        // ----------------------------
        stage('Checkout Code') {
            steps {
                git 'https://github.com/org/repo.git'
            }
        }

        // ----------------------------
        stage('Fetch Secrets from Vault') {
            steps {
                withCredentials([string(credentialsId: 'vault-token', variable: 'VAULT_TOKEN')]) {
                    sh """
                        export DB_PASSWORD=$(vault kv get -field=password secret/hotel/db)
                        export DB_USER=$(vault kv get -field=username secret/hotel/db)
                        echo "Secrets fetched from Vault"
                    """
                }
            }
        }

        // ----------------------------
        stage('Run Unit Tests') {
            steps {
                echo "Running unit tests"
                sh 'mvn clean test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        // ----------------------------
        stage('Dependency Scan') {
            steps {
                echo "Running OWASP Dependency Check"
                sh 'mvn org.owasp:dependency-check-maven:check'
            }
        }

        // ----------------------------
        stage('SonarQube Analysis') {
            steps {
                echo "Running SonarQube analysis"
                withSonarQubeEnv("${SONARQUBE}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        // ----------------------------
        stage('Quality Gate') {
            steps {
                echo "Waiting for SonarQube Quality Gate"
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // ----------------------------
        stage('IaC Security Scan') {
            steps {
                echo "Scanning Terraform code with Checkov"
                sh 'checkov -d terraform/'
                // Optionally, also run tfsec
                // sh 'tfsec terraform/'
            }
        }

        // ----------------------------
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image"
                sh "docker build -t $REGISTRY/$IMAGE:$BUILD_NUMBER ."
            }
        }

        // ----------------------------
        stage('Trivy Image Scan') {
            steps {
                echo "Scanning Docker image with Trivy"
                sh "trivy image --exit-code 1 --severity HIGH,CRITICAL $REGISTRY/$IMAGE:$BUILD_NUMBER || true"
            }
        }

        // ----------------------------
        stage('Push to ECR') {
            steps {
                echo "Pushing Docker image to ECR"
                sh """
                aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin $REGISTRY
                docker push $REGISTRY/$IMAGE:$BUILD_NUMBER
                """
            }
        }

        // ----------------------------
        stage('Update Helm Chart for ArgoCD') {
            steps {
                echo "Updating Helm chart with new image tag"
                sh """
                sed -i 's/tag:.*/tag: $BUILD_NUMBER/' helm/values.yaml
                git add helm/values.yaml
                git commit -m "Update image tag to $BUILD_NUMBER"
                git push
                """
            }
        }

    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed! Check logs, SonarQube, Trivy, or IaC scans for details."
        }
    }
}


Pipeline Flow (Interview Explanation)

Checkout Code: Pulls latest code from GitHub.

Vault Integration: Fetches secrets like DB username/password securely.

Unit Tests: mvn clean test runs all unit tests; results archived.

Dependency Scan: OWASP Dependency Check scans Maven dependencies for CVEs.

SonarQube Analysis: Static code analysis, checking quality, security, and coverage.

Quality Gate: Pipeline fails if code quality thresholds are not met.

IaC Security Scan: Checkov/tfsec validates Terraform infrastructure for misconfigurations.

Docker Build: Builds container with compiled JAR.

Trivy Scan: Scans Docker image for vulnerabilities; prevents risky images from reaching production.

Push to ECR: Stores Docker image in AWS ECR.

Update Helm Chart / GitOps: ArgoCD detects Helm changes and deploys to EKS cluster.
