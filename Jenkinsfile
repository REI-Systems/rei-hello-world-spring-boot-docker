node {
    def app

    stage('Configure') {
        env.PATH = "${tool 'maven-3.5'}/bin:${env.PATH}"
        env.PATH = "${tool 'JDK8'}/bin:${env.PATH}"
        version = '1.0.' + env.BUILD_NUMBER
        currentBuild.displayName = version
    }

    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout scm
    }

    stage('Build and Unit test') {
        /* Build JAR file */

        sh 'mvn clean install cobertura:cobertura -Dcobertura.report.format=xml'
    }

    stage('Build image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */

        app = docker.build("ykhadilkar/rei-hello-world-spring-boot-docker")
    }

    stage('Test image') {
        /* Ideally, we would run a test framework against our image. */

        app.inside {
            sh 'echo "Tests passed"'
        }
    }

    stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }

    stage('Deploy Image'){
        /* Deploy docker image from docker hub*/
        sh 'docker pull ykhadilkar/rei-hello-world-spring-boot-docker:latest'
        sh 'docker stop ykhadilkar/rei-hello-world-spring-boot-docker:latest || true && docker rm ykhadilkar/rei-hello-world-spring-boot-docker:latest || true'
        sh 'docker run -p 8080:8080 -d --restart=always ykhadilkar/rei-hello-world-spring-boot-docker:latest'
    }
}