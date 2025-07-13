node {

    def mavenHome
    def mavenCMD
    def docker
    def dockerCMD
    def tagName // Will now be dynamic for better versioning

    stage('Prepare Environment') {
        echo 'Initializing all the variables'
        mavenHome = tool name: 'maven' , type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        docker = tool name: 'docker' , type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        dockerCMD = "${docker}/bin/docker"
        // Using Jenkins's built-in BUILD_NUMBER for a dynamic tag for Docker images.
        // This helps in tracking which image corresponds to which Jenkins build.
        tagName = "${BUILD_NUMBER}"
    }

    stage('Git Code Checkout') {
        try {
            echo 'Checking out the code from git repository'
            git 'https://github.com/shubhamkushwah123/star-agile-insurance-project.git'
        } catch (Exception e) {
            echo "Exception occurred in Git Code Checkout Stage: ${e.getMessage()}"
            currentBuild.result = "FAILURE"
            // Send email notification on failure for this critical stage
            emailext body: '''Dear All,
The Jenkins job ${JOB_NAME} has been failed during Git Code Checkout. Please check the build logs immediately.
${BUILD_URL}''', subject: "Jenkins Job ${JOB_NAME} ${BUILD_NUMBER} - FAILED (Git Checkout)", to: 'shubham@gmail.com'
            error "Git Code Checkout failed. Terminating pipeline." // Terminate pipeline on critical failure
        }
    }

    stage('Build the Application') {
        try {
            echo "Cleaning... Compiling...Testing... Packaging..."
            // Execute Maven build. This will now include Selenium dependencies due to pom.xml changes.
            sh "${mavenCMD} clean package"
        } catch (Exception e) {
            echo "Exception occurred in Build Application Stage: ${e.getMessage()}"
            currentBuild.result = "FAILURE"
            emailext body: '''Dear All,
The Jenkins job ${JOB_NAME} has been failed during Application Build. Please check the build logs immediately.
${BUILD_URL}''', subject: "Jenkins Job ${JOB_NAME} ${BUILD_NUMBER} - FAILED (Build)", to: 'shubham@gmail.com'
            error "Application Build failed. Terminating pipeline."
        }
    }

    stage('Publish Test Reports') {
        // Corrected reportDir to use WORKSPACE variable for portability.
        // Assumes surefire-reports are generated in target/surefire-reports within your workspace.
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: "${WORKSPACE}/target/surefire-reports", reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
    }

    stage('Containerize the Application') {
        try {
            echo 'Creating Docker image'
            // Builds the Docker image with the dynamic tag name
            sh "${dockerCMD} build -t shubhamkushwah123/insure-me:${tagName} ."
        } catch (Exception e) {
            echo "Exception occurred in Containerization Stage: ${e.getMessage()}"
            currentBuild.result = "FAILURE"
            emailext body: '''Dear All,
The Jenkins job ${JOB_NAME} has been failed during Docker Image Creation. Please check the build logs immediately.
${BUILD_URL}''', subject: "Jenkins Job ${JOB_NAME} ${BUILD_NUMBER} - FAILED (Containerize)", to: 'shubham@gmail.com'
            error "Containerization failed. Terminating pipeline."
        }
    }

    //
