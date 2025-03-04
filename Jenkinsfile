// Created scripted jenkinsfile
node {
  checkout scm
    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
        }
    }

    stage('Test') {
        docker.image('qnib/pytest').inside {
            try {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            } finally {
                junit 'test-reports/results.xml'
            }
        }
    }

  stage('Manual Approval') {
        def userInput = input(
            id: 'userInput', 
            message: 'Lanjutkan ke tahap Deploy?', 
            parameters: [
                [$class: 'BooleanParameterDefinition', name: 'Proceed', defaultValue: true]
            ]
        )
        
        if (!userInput) {
            error('Pipeline dihentikan oleh pengguna.')
        }
    }
    stage('Deploy') {
        docker.image('python:3.9').inside('-u root') {
            sh 'pip install pyinstaller'
            sh 'pyinstaller --onefile sources/add2vals.py'

            sleep(time: 1, unit: 'MINUTES')
            echo 'Pipeline has finished successfully.'
        }
        if (currentBuild.result == 'SUCCESS') {
            archiveArtifacts 'dist/add2vals'
        }
    }
}
