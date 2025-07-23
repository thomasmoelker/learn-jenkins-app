pipeline {
	agent any

	environment {
		NETLIFY_SITE_ID = '0cb2e29f-3575-4bc1-a0c5-2a0d650b0e97'
		NETLIFY_AUTH_TOKEN = credentials('netlify-token')
	}

    stages {
		stage('Build') {
			agent {
				docker {
					image 'node:18-alpine'
					reuseNode true
				}
			}
			steps {
				sh '''
					echo "Test for autamatic deployment"
					ls -la
					node --version
					npm --version
					npm ci
					npm run build
					ls -la
				'''
            }
        }
        stage('Run Tests'){
			parallel {
				stage('Unit test') {
					agent {
						docker {
							image 'node:18-alpine'
						reuseNode true
					}
					}
					steps {
						sh '''
						test -f build/index.html
						npm test
						'''
					}
					post {
						always {
							junit 'jest-results/junit.xml'
						}
					}
				}
				stage('E2E') {
					agent {
						docker {
							image 'mcr.microsoft.com/playwright:v1.54.0-noble'
							reuseNode true
						}
					}
					steps {
						sh '''
							npm install serve
							node_modules/.bin/serve -s build &
							sleep 10
							npx playwright test --reporter=html
						'''
					}
					post{
						always {
							publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report Local', reportTitles: '', useWrapperFileDirectly: true])
						}
					}
				}
			}
		}
		stage('Deploy Staging') {
			agent {
				docker {
					image 'mcr.microsoft.com/playwright:v1.54.0-noble'
						reuseNode true
					}
			}
			environment {
				CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
			}

			steps {
				sh '''
						npm install netlify-cli
						npm install node-jq
						node_modules/.bin/netlify --version
						echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
						node_modules/.bin/netlify status
						node_modules/.bin/netlify deploy --dir=build --no-build --json > deploy-output.json
						CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r ".deploy_url" deploy-output.json)
						npx playwright test --reporter=html
					'''
				}
			post{
				always {
					publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report Staging', reportTitles: '', useWrapperFileDirectly: true])
					}
				}

		}
        stage('Deploy Production') {
			environment {
				CI_ENVIRONMENT_URL = 'https://gleaming-douhua-144faa.netlify.app'
			}
			agent {
				docker {
					image 'mcr.microsoft.com/playwright:v1.54.0-noble'
						reuseNode true
					}
				}
				steps {
				sh '''
						npm --version
						npm install netlify-cli
						node_modules/.bin/netlify --version
						echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
						node_modules/.bin/netlify status
						node_modules/.bin/netlify deploy --dir=build --prod --no-build
						npx playwright test --reporter=html
					'''
				}
				post{
				always {
					publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report Production', reportTitles: '', useWrapperFileDirectly: true])
					}
				}

		}
	}
}



