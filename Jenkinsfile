pipeline {
    agent any

    tools { nodejs 'NodeJS' }

    options {
        ansiColor('xterm')
    }

    parameters {
        string(name: 'SPEC', defaultValue:'cypress/e2e/1-getting-started/todo.cy.js', description: 'Enter the cypress script path that you want to execute')
        choice(name: 'BROWSER', choices:['electron', 'chrome', 'edge', 'firefox'], description: 'Select the browser to be used in your cypress tests')
        booleanParam(name: 'skip_sonar', defaultValue: true, description: 'Set to true to skip the test stage')
    }

    stages {
        stage('Build/Deploy app to staging-') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                        configName: 'staging',
                        transfers: [
                            sshTransfer(
                            cleanRemote: false,
                            excludes: 'node_modules/,cypress/,**/*.yml,mochawesome-report/,.scannerwork/',
                            execCommand: 'cd /var/www/html && npm install && pm2 restart npm serve.js || pm2 start npm serve.js',
                            execTimeout: 120000,
                            flatten: false,
                            makeEmptyDirs: false,
                            noDefaultExcludes: false,
                            patternSeparator: '[, ]+',
                            remoteDirectory: '',
                            remoteDirectorySDF: false,
                            removePrefix: '',
                            sourceFiles: '**/*')],
                        usePromotionTimestamp: false,
                        useWorkspaceInPromotion: false,
                        failOnError: true,
                        continueOnError:false,

                    verbose: true)])
            } }

        stage('Run automated tests') {
            steps {
                echo 'Running automated tests'
                sh 'npm prune'
                sh 'npm cache clean --force'
                sh 'npm i'
                sh 'npm install -g cypress --force'  //forçar instalação do cypress
                sh 'npm install --save-dev mochawesome mochawesome-merge mochawesome-report-generator'
                sh 'npm run e2e_tests'
            }

            post {
                success {
                    publishHTML(
                        target : [
                            allowMissing: false,
                            alwaysLinkToLastBuild: true,
                            keepAll: true,
                            reportDir: 'mochawesome-report',
                            reportFiles: 'mochawesome.html',
                            reportName: 'My Reports',
                            reportTitles: 'The Report'])
                }
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
        stage('Quality Gate') {
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
                timeout(activity: true, time: 5) {
                    input 'Proceed to production?'
                }
            }
        }
        stage('Release to production') {
            steps {
                echo 'Releasing to production'
            }
        }
        }
    }
