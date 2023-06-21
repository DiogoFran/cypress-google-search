pipeline {
    agent any

    tools { nodejs 'NodeJS' }

    parameters {
        string(name: 'SPEC', defaultValue:'cypress/e2e/1-getting-started/todo.cy.js', description: 'Enter the cypress script path that you want to execute')
        choice(name: 'BROWSER', choices:['electron', 'chrome', 'edge', 'firefox'], description: 'Select the browser to be used in your cypress tests')
        booleanParam(name: 'skip_sonar', defaultValue: true, description: 'Set to true to skip the test stage')
    }

    stages {
        stage('Run automated tests') {
            steps {
                echo 'Running automated tests'
                sh 'npm prune'
                sh 'npm cache clean --force'
                sh 'npm i'
                sh 'npm install -g cypress --force'  //forçar instalação do cypress
                sh 'npm install --save-dev mochawesome mochawesome-merge mochawesome-report-generator'
                sh 'npx cypress open'
            }
        }
        stage('SonarQube analysis') {
            when { expression { params.skip_sonar != true } }
            steps {
                    script {
                        scannerHome = tool 'sonar-scanner'
                    }
                    withSonarQubeEnv('CloudSonarDiogoFran') { // Replace this name by the one you setup in the SonarQube Jenkins configuration (step 2)
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
            }
        }
        stage('JMeter Test') {
            when { expression { params.skip_sonar != true } }
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        stage('Perform manual testing') {
            steps {
                timeout(activity: true, time: 2,unit: 'DAYS') {
                    input 'Proceed to production?'
                }
            }
        }
        }
    }
