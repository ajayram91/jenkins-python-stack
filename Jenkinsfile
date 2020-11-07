stage('test') {
    agent {
        docker {
            image 'python:alpine'
        }
    }
    steps {
        withEnv(["HOME=${env.WORKSPACE}"]) {
            sh 'python -m pytest -v --junit-xml results.xml src/appTest.py'
        }
    }
    post {
        always {
            junit 'results.xml'
        }
    }
}
