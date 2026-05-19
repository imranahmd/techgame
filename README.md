# TechgameListingWebApp

## Description

**Board Game Database Full-Stack Web Application.**
This web application displays lists of board games and their reviews. While anyone can view the board game lists and reviews, they are required to log in to add/ edit the board games and their reviews. The 'users' have the authority to add board games to the list and add reviews, and the 'managers' have the authority to edit/ delete the reviews on top of the authorities of users.  

## Technologies

- Java
- Spring Boot
- Amazon Web Services(AWS) EC2
- Thymeleaf
- Thymeleaf Fragments
- HTML5
- CSS
- JavaScript
- Spring MVC
- JDBC
- H2 Database Engine (In-memory)
- JUnit test framework
- Spring Security
- Twitter Bootstrap
- Maven

## Features

- Full-Stack Application
- UI components created with Thymeleaf and styled with Twitter Bootstrap
- Authentication and authorization using Spring Security
  - Authentication by allowing the users to authenticate with a username and password
  - Authorization by granting different permissions based on the roles (non-members, users, and managers)
- Different roles (non-members, users, and managers) with varying levels of permissions
  - Non-members only can see the boardgame lists and reviews
  - Users can add board games and write reviews
  - Managers can edit and delete the reviews
- Deployed the application on AWS EC2
- JUnit test framework for unit testing
- Spring MVC best practices to segregate views, controllers, and database packages
- JDBC for database connectivity and interaction
- CRUD (Create, Read, Update, Delete) operations for managing data in the database
- Schema.sql file to customize the schema and input initial data
- Thymeleaf Fragments to reduce redundancy of repeating HTML elements (head, footer, navigation)

## How to Run

1. Clone the repository
2. Open the project in your IDE of choice
3. Run the application
4. To use initial user data, use the following credentials.
  - username: bugs    |     password: bunny (user role)
  - username: daffy   |     password: duck  (manager role)
5. You can also sign-up as a new user and customize your role to play with the application! 😊




https://github.com/techworldwithmurali/devops-real-time-projects-bootcamp/tree/main/Tools%20Installation%20Steps/Jfrog%20Artifactory


https://github.com/saikiranpi/Mastering-DevSecOps/blob/Master/Day%2033%20Jenkins-Part-1/Jenkinsfile


https://github.com/jfrog/jenkins-jfrog-plugin/blob/main/.jfrog-pipelines/pipelines.release.yml


https://github.com/devopswithprashant/guide/blob/main/Install/postgresql-install-doc.md

https://github.com/ravdy/DevOps/blob/master/Artifactory/Setup_Artifactory.md

https://github.com/jaiswaladi246/Petclinic/blob/main/Jenkinsfile

https://github.com/ValaxyTech/DevOpsDemos/blob/master/Jenkins/S3_Artifact_for_Jenkins.md


pipeline {
    agent any

    environment {
        EFS_PATH = "/mnt/upsc-wp-dev/upsc-website-backend"
        GIT_CREDENTIALS_ID = "8f0ed0e5-d4b8-4919-ab40-5620cf6ec7d1"
        REPO_URL = "https://openforge.gov.in/plugins/git/upsc-projects/upsc-website-backend.git"
        RUNTIME_PATH = "/opt/upsc-website-backend"
    }

    parameters {
        string(
            name: 'SOURCE_BRANCH',
            defaultValue: 'dev',
            description: 'Source branch to deploy from (dev / uat / main)'
        )
    }

    stages {

        stage("Checkout Source Branch") {
            steps {
                echo "Checking out ${params.SOURCE_BRANCH} branch..."

                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${params.SOURCE_BRANCH}"]],
                    userRemoteConfigs: [[
                        url: "${REPO_URL}",
                        credentialsId: "${GIT_CREDENTIALS_ID}"
                    ]]
                ])
            }
        }

        stage("Create & Push Deployment Branch") {
            steps {

                script {
                    env.DEPLOY_BRANCH = "${params.SOURCE_BRANCH}-release-${env.BUILD_NUMBER}"
                }

                withCredentials([usernamePassword(
                    credentialsId: "${GIT_CREDENTIALS_ID}",
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {

                    sh '''
                    git config user.name "$GIT_USER"
                    git config user.email "$GIT_USER@openforge.gov.in"

                    # create release branch from current checked-out commit
                    git checkout -B ${DEPLOY_BRANCH}

                    # push branch using credentials
                    git push https://$GIT_USER:$GIT_PASS@openforge.gov.in/plugins/git/upsc-projects/upsc-website-backend.git ${DEPLOY_BRANCH}
                    '''
                }
            }
        }

        stage('Sync Code to EFS') {
            steps {
                sh '''
                echo "Syncing code to EFS path: ${EFS_PATH}"

                rsync -avz \
                  --exclude 'wp-content/uploads/' \
                  --no-o --no-g \
                  --omit-dir-times \
                  ./ ${EFS_PATH}/
                '''
            }
        }

        stage('Copy Runtime Config') {
            steps {
                sh '''
                echo "Copying runtime configs..."

                cp -rvf ${RUNTIME_PATH}/wp-config.php ${EFS_PATH}/wp-config.php
                cp -rvf ${RUNTIME_PATH}/.htaccess ${EFS_PATH}/.htaccess
                '''
            }
        }
    }

    post {
        success {
            echo "======================================"
            echo "SUCCESS: Code deployed from ${params.SOURCE_BRANCH}"
            echo "Release branch created: ${env.DEPLOY_BRANCH}"
            echo "======================================"
        }

        failure {
            echo "======================================"
            echo "FAILED: Deployment from ${params.SOURCE_BRANCH}"
            echo "======================================"
        }
    }
}

