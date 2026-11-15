
# Jenkins Shared Library – Groovy Examples

## 1. Introduction

Jenkins Shared Libraries allow us to write reusable pipeline code and share it across multiple Jenkins pipelines.

Instead of writing the same code repeatedly in Jenkinsfiles, we can create reusable Groovy functions inside a Shared Library.

### Benefits

- Code reusability
- Easy maintenance
- Reduced Jenkinsfile complexity
- Standardized CI/CD pipelines
- Better pipeline management

---

## 2. Jenkins Shared Library Structure

```text
jenkins-shared-libraries/
│
├── vars/
│   ├── hello.groovy
│   ├── greet.groovy
│   ├── buildApp.groovy
│   ├── gitCheckout.groovy
│   ├── dockerBuild.groovy
│   ├── dockerPush.groovy
│   ├── deployApp.groovy
│   └── sendNotification.groovy
│
├── src/
│   └── org/
│       └── example/
│           └── Utilities.groovy
│
└── resources/
    └── config/
        └── app-config.yaml
```

### Directory Explanation

| Directory | Description |
|---|---|
| vars/ | Contains reusable pipeline functions |
| src/ | Contains reusable Groovy classes |
| resources/ | Contains configuration files and templates |

---

# 3. Basic Groovy Examples

## Example 1: hello.groovy

Print a simple message.

**File:** `vars/hello.groovy`

```groovy
def call() {
    echo "Hello from Jenkins Shared Library!"
}
```

**Jenkinsfile:**

```groovy
@Library('shared@main') _

pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                script {
                    hello()
                }
            }
        }
    }
}
```

---

## Example 2: greet.groovy

Pass parameters to a shared function.

**File:** `vars/greet.groovy`

```groovy
def call(String name) {
    echo "Hello ${name}, welcome to Jenkins!"
}
```

**Jenkinsfile:**

```groovy
@Library('shared@main') _

pipeline {
    agent any

    stages {
        stage('Greeting') {
            steps {
                script {
                    greet('Kunal')
                }
            }
        }
    }
}
```

**Output:**

```text
Hello Kunal, welcome to Jenkins!
```

---

## Example 3: buildApp.groovy

Execute a Maven build.

**File:** `vars/buildApp.groovy`

```groovy
def call() {
    echo "Starting Maven Build"

    sh 'mvn clean package'

    echo "Build completed successfully!"
}
```

**Jenkinsfile:**

```groovy
@Library('shared@main') _

pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                script {
                    buildApp()
                }
            }
        }
    }
}
```

---

## Example 4: gitCheckout.groovy

Checkout a Git repository using a reusable function.

**File:** `vars/gitCheckout.groovy`

```groovy
def call(String repoUrl, String branch) {

    echo "Checking out ${branch}"

    git branch: branch,
        url: repoUrl

    echo "Git checkout completed!"
}
```

**Jenkinsfile:**

```groovy
@Library('shared@main') _

pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                script {
                    gitCheckout(
                        'https://github.com/example/demo.git',
                        'main'
                    )
                }
            }
        }
    }
}
```

---

# 4. Intermediate Groovy Examples

## Example 5: dockerBuild.groovy

Build a Docker image.

**File:** `vars/dockerBuild.groovy`

```groovy
def call(String imageName, String tag) {

    echo "Building Docker Image"

    sh "docker build -t ${imageName}:${tag} ."

    echo "Docker image created successfully!"
}
```

**Jenkinsfile:**

```groovy
@Library('shared@main') _

pipeline {
    agent any

    stages {
        stage('Docker Build') {
            steps {
                script {
                    dockerBuild('myapp', 'v1')
                }
            }
        }
    }
}
```

---

## Example 6: dockerPush.groovy

Push a Docker image to Docker Hub.

**File:** `vars/dockerPush.groovy`

```groovy
def call(String imageName, String tag) {

    withCredentials([
        usernamePassword(
            credentialsId: 'dockerhub-credentials',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
        )
    ]) {

        sh '''
            echo "$DOCKER_PASS" | docker login \
                -u "$DOCKER_USER" --password-stdin

            docker push "$DOCKER_USER/IMAGE_PLACEHOLDER"
            docker logout
        '''.replace(
            'IMAGE_PLACEHOLDER',
            "${imageName}:${tag}"
        )
    }
}
```

**Jenkinsfile:**

```groovy
@Library('shared@main') _

pipeline {
    agent any

    stages {
        stage('Docker Push') {
            steps {
                script {
                    dockerPush('myapp', 'v1')
                }
            }
        }
    }
}
```

Note: Configure the Docker Hub credentials in Jenkins before running this example. For a production implementation, use the fully qualified image name, including the Docker Hub username.

---

## Example 7: sendNotification.groovy

Print a notification in Jenkins console output.

**File:** `vars/sendNotification.groovy`

```groovy
def call(String message) {

    echo "================================"
    echo "JENKINS NOTIFICATION"
    echo "================================"

    echo "${message}"

    echo "================================"
}
```

**Jenkinsfile:**

```groovy
@Library('shared@main') _

pipeline {
    agent any

    stages {
        stage('Notification') {
            steps {
                script {
                    sendNotification(
                        'Build completed successfully!'
                    )
                }
            }
        }
    }
}
```

---

## Example 8: deployApp.groovy

Deploy an application to a remote server using SSH.

**File:** `vars/deployApp.groovy`

```groovy
def call(String server, String appName) {

    echo "Deploying ${appName}"

    sshagent(['server-ssh-key']) {

        sh """
            ssh -o StrictHostKeyChecking=no \
            ec2-user@${server} \
            'sudo systemctl restart ${appName}'
        """
    }

    echo "Deployment completed!"
}
```

**Jenkinsfile:**

```groovy
@Library('shared@main') _

pipeline {
    agent any

    stages {
        stage('Deploy') {
            steps {
                script {
                    deployApp(
                        '10.0.0.10',
                        'myapp'
                    )
                }
            }
        }
    }
}
```

Requirements:
- Install the SSH Agent plugin.
- Configure an SSH private-key credential.
- Ensure the remote user has permission to restart the application.

---

# 5. Advanced Groovy Examples

## Example 9: buildApp.groovy using Map

A Map allows us to pass multiple named parameters to a function.

**File:** `vars/buildApp.groovy`

```groovy
def call(Map config) {

    echo "Application: ${config.appName}"
    echo "Build Tool: ${config.tool}"
    echo "Version: ${config.version}"

    if (config.tool == 'maven') {

        sh 'mvn clean package'

    } else if (config.tool == 'gradle') {

        sh './gradlew clean build'

    } else {

        error "Unsupported build tool: ${config.tool}"
    }
}
```

**Jenkinsfile:**

```groovy
@Library('shared@main') _

pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                script {
                    buildApp(
                        appName: 'payment-service',
                        tool: 'maven',
                        version: '1.0'
                    )
                }
            }
        }
    }
}
```

---

## Example 10: deployApp.groovy with Environment Selection

Deploy an application based on the selected environment.

**File:** `vars/deployApp.groovy`

```groovy
def call(String environment) {

    if (environment == 'dev') {

        echo "Deploying to DEV"

    } else if (environment == 'stage') {

        echo "Deploying to STAGE"

    } else if (environment == 'prod') {

        echo "Deploying to PRODUCTION"

    } else {

        error "Invalid environment: ${environment}"
    }
}
```

**Jenkinsfile:**

```groovy
@Library('shared@main') _

pipeline {
    agent any

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'stage', 'prod'],
            description: 'Select deployment environment'
        )
    }

    stages {
        stage('Deploy') {
            steps {
                script {
                    deployApp(params.ENVIRONMENT)
                }
            }
        }
    }
}
```

---

# 6. Using src/ Directory

The `src/` directory is used to create reusable Groovy classes.

## Example 11: Utilities.groovy

**File:** `src/org/example/Utilities.groovy`

```groovy
package org.example

class Utilities {

    static String greet(String name) {
        return "Hello ${name}!"
    }

    static int add(int a, int b) {
        return a + b
    }
}
```

**Jenkinsfile:**

```groovy
@Library('shared@main') _

import org.example.Utilities

pipeline {
    agent any

    stages {
        stage('Utilities') {
            steps {
                script {
                    echo Utilities.greet('Kunal')

                    def result = Utilities.add(10, 20)

                    echo "Addition: ${result}"
                }
            }
        }
    }
}
```

**Output:**

```text
Hello Kunal!
Addition: 30
```

---

# 7. Complete CI/CD Pipeline Using Shared Library

This example combines multiple shared library functions.

### Jenkinsfile

```groovy
@Library('shared@main') _

pipeline {

    agent {
        label 'charli'
    }

    environment {
        APP_NAME = 'myapp'
        IMAGE_TAG = 'v1'
    }

    stages {

        stage('Hello') {
            steps {
                script {
                    hello()
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    buildApp()
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    dockerBuild(
                        env.APP_NAME,
                        env.IMAGE_TAG
                    )
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    dockerPush(
                        env.APP_NAME,
                        env.IMAGE_TAG
                    )
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}
```

---

# 8. Important Groovy Concepts

| Syntax | Description |
|---|---|
| `def call()` | Defines a callable shared library function |
| `def call(String name)` | Function with a string parameter |
| `def call(Map config)` | Function accepting named parameters |
| `echo` | Prints messages in Jenkins console |
| `sh` | Executes shell commands |
| `if / else` | Conditional execution |
| `return` | Returns a value |
| `error()` | Fails the Jenkins pipeline |
| `withCredentials()` | Accesses Jenkins credentials securely |
| `try / catch` | Handles exceptions |

---

# 9. Jenkins Shared Library Configuration

1. Open Jenkins Dashboard.
2. Navigate to Manage Jenkins.
3. Open System.
4. Find Global Pipeline Libraries.
5. Add a new library.

Configure the following:

| Field | Value |
|---|---|
| Name | shared |
| Default version | main |
| Retrieval method | Modern SCM |
| Source Code Management | Git |
| Project Repository | Your GitHub repository URL |

Use the following annotation in your Jenkinsfile:

```groovy
@Library('shared@main') _
```

---

# 10. Best Practices

- Keep reusable functions inside `vars/`.
- Keep reusable Groovy classes inside `src/`.
- Use meaningful function and file names.
- Avoid hardcoding credentials.
- Use Jenkins Credentials for passwords and tokens.
- Keep Jenkinsfiles simple.
- Use parameters to make functions reusable.
- Test shared library changes before using them in production.
- Use version control to manage library changes.

---

# 11. Practice Exercises

1. Create `calculator.groovy` to perform addition, subtraction, multiplication and division.
2. Create `testApp.groovy` to execute Maven unit tests.
3. Create `dockerCleanup.groovy` to remove unused Docker images.
4. Create `backup.groovy` to create timestamped backups.
5. Create `deploy.groovy` to deploy applications to DEV, QA and PROD.

---

# Conclusion

Jenkins Shared Libraries help us build reusable, maintainable and standardized CI/CD pipelines.

By learning Groovy functions, parameters, Maps and classes, we can create a complete CI/CD framework and reuse it across multiple Jenkins projects.
