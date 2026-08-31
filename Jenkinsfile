pipeline {

    agent any

    parameters {

        choice(
            name: 'OPERATION',
            choices: [
                'create',
                'list',
                'merge',
                'rebase',
                'delete'
            ],
            description: 'Select the Git operation'
        )

        string(
            name: 'BRANCH_NAME',
            defaultValue: '',
            description: 'Branch name for create/delete operation'
        )

        string(
            name: 'SOURCE_BRANCH',
            defaultValue: '',
            description: 'Source branch for merge/rebase'
        )

        string(
            name: 'TARGET_BRANCH',
            defaultValue: 'main',
            description: 'Target branch for merge/rebase'
        )
    }

    stages {

        stage('Checkout Repository') {
            steps {
                checkout scm
            }
        }

        stage('Git Operation') {
            steps {

                sh '''
                    git config user.name "Jenkins"
                    git config user.email "jenkins@example.com"

                    echo "Selected Operation: ${OPERATION}"

                    case "${OPERATION}" in

                        create)

                            if [ -z "${BRANCH_NAME}" ]; then
                                echo "ERROR: BRANCH_NAME is required"
                                exit 1
                            fi

                            echo "Creating branch: ${BRANCH_NAME}"

                            git fetch --all

                            git checkout -b "${BRANCH_NAME}"

                            git push -u origin "${BRANCH_NAME}"

                            ;;

                        list)

                            echo "Listing all branches..."

                            echo "----- Local Branches -----"
                            git branch

                            echo "----- Remote Branches -----"
                            git branch -r

                            echo "----- All Branches -----"
                            git branch -a

                            ;;

                        merge)

                            if [ -z "${SOURCE_BRANCH}" ] || [ -z "${TARGET_BRANCH}" ]; then
                                echo "ERROR: SOURCE_BRANCH and TARGET_BRANCH are required"
                                exit 1
                            fi

                            echo "Merging ${SOURCE_BRANCH} into ${TARGET_BRANCH}"

                            git fetch --all

                            git checkout "${TARGET_BRANCH}"

                            git pull origin "${TARGET_BRANCH}"

                            git merge "${SOURCE_BRANCH}"

                            git push origin "${TARGET_BRANCH}"

                            ;;

                        rebase)

                            if [ -z "${SOURCE_BRANCH}" ] || [ -z "${TARGET_BRANCH}" ]; then
                                echo "ERROR: SOURCE_BRANCH and TARGET_BRANCH are required"
                                exit 1
                            fi

                            echo "Rebasing ${SOURCE_BRANCH} onto ${TARGET_BRANCH}"

                            git fetch --all

                            git checkout "${SOURCE_BRANCH}"

                            git pull origin "${SOURCE_BRANCH}"

                            git rebase "origin/${TARGET_BRANCH}"

                            git push --force-with-lease origin "${SOURCE_BRANCH}"

                            ;;

                        delete)

                            if [ -z "${BRANCH_NAME}" ]; then
                                echo "ERROR: BRANCH_NAME is required"
                                exit 1
                            fi

                            if [ "${BRANCH_NAME}" = "main" ]; then
                                echo "ERROR: Refusing to delete main branch"
                                exit 1
                            fi

                            echo "Deleting branch: ${BRANCH_NAME}"

                            git push origin --delete "${BRANCH_NAME}"

                            ;;

                        *)

                            echo "ERROR: Invalid operation"
                            exit 1

                            ;;

                    esac
                '''
            }
        }
    }

    post {

        success {
            echo "Git operation completed successfully."
        }

        failure {
            echo "Git operation failed."

            // Email notification
            emailext(
                subject: "Jenkins FAILURE: ${JOB_NAME} #${BUILD_NUMBER}",
                body: """
Jenkins Git operation failed.

Job: ${JOB_NAME}
Build: ${BUILD_NUMBER}
Operation: ${OPERATION}
Build URL: ${BUILD_URL}

Please check the Jenkins console log.
                """,
                to: "tanushirana875@gmail.com"
            )

            // Slack notification
//             slackSend(
//                 channel: '#jenkins',
//                 message: """
// Jenkins Git operation FAILED.

// Job: ${JOB_NAME}
// Build: #${BUILD_NUMBER}
// Operation: ${OPERATION}
// Build URL: ${BUILD_URL}
//                 """
//             )
        }
    }
}
