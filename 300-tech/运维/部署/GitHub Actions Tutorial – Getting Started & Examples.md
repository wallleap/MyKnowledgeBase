---
title: GitHub Actions Tutorial – Getting Started & Examples
date: 2024-08-30T09:53:13+08:00
updated: 2024-08-30T09:53:49+08:00
dg-publish: false
source: https://spacelift.io/blog/github-actions-tutorial
---

In this post, we’ll explore various concepts related to GitHub Actions and how they enable CI/CD in the application development lifecycle. We’ll also work through an example to understand and implement these concepts using GitHub Actions.

1. [What is GitHub Actions?](https://spacelift.io/blog/github-actions-tutorial#what-is-github-actions)
2. [GitHub Actions key concepts and features](https://spacelift.io/blog/github-actions-tutorial#github-actions-key-concepts-and-features)
3. [Tutorial: How to create GitHub Actions workflow](https://spacelift.io/blog/github-actions-tutorial#tutorial-how-to-create-a-github-actions-workflow)
4. [GitHub Actions examples](https://spacelift.io/blog/github-actions-tutorial#github-actions-examples)

## What is Github Actions?[](https://spacelift.io/blog/github-actions-tutorial#what-is-github-actions)

GitHub Actions is a continuous integration and continuous delivery/deployment (CI/CD) platform that automates your software development workflows. It allows you to build, test, and deploy software source code directly from your GitHub repository by creating custom workflows or pipelines. With various configuration options for triggers based on commits and merges, GitHub Actions is a good choice for GitOps-based workflows.

### How does GitHub Actions work?

![](https://cdn.wallleap.cn/img/pic/illustration/202408300955916.png?imageSlim)


GitHub Actions workflows are configured using [YAML files](https://spacelift.io/blog/yaml) that define the sequence of tasks or actions to be executed when triggered by events like code pushes, pull requests, and releases. 

Actions are reusable units of code that perform specific tasks, such as setting up dependencies, running tests, and deploying to a cloud provider or on-premise servers. You can create and publish custom Actions for your specific requirements.

GitHub provides virtual machines, a.k.a. runners, to run your workflows on Linux, Windows, and macOS environments. You can also host your own self-hosted runners. Runners support any programming language, platform, and cloud provider, making GitHub flexible for various projects. 

GitHub Actions seamlessly integrates with other GitHub features, such as Issues, PRs, and Marketplace, allowing you to create automated workflows based on events in your repository.

### Is GitHub Actions easy to learn?

GitHub Actions is generally considered easy to learn, especially for developers already familiar with GitHub and basic scripting concepts. It provides a user-friendly entry point for automation, allowing developers to quickly set up basic [CI/CD pipelines](https://spacelift.io/blog/ci-cd-pipeline) and other workflows without extensive DevOps knowledge. As with any tool, mastering more advanced features and complex use cases will require additional time and practice.

## GitHub Actions key concepts and features[](https://spacelift.io/blog/github-actions-tutorial#github-actions-key-concepts-and-features)

Before we delve into examples of GitHub Actions, the concepts outlined below provide an essential overview and will help you understand the topic more easily.

### 1. GitHub Actions workflows

In GitHub Actions, a workflow is an automated process defined by a YAML file. It is usually placed in the .github/workflows directory of any repository. It describes one or more jobs (related to build, test, and deploy) that can be triggered by events like code pushes, pull requests, releases, or scheduled times. You can use workflow files to configure pipelines to take a predefined series of actions on the code that has just been committed.

### 2. GitHub Actions events

GitHub Actions events are specific activities that trigger a workflow run. These events serve as trigger points for workflows, which help automate the build and deployment processes. Common event triggers include push, pull_request, schedule, and workflow_dispatch. Additionally, you can fine-tune these triggers by specifying further details such as branch names, commit messages, and more.

### 3. GitHub Actions runners

Runners in GitHub Actions are virtual machines that execute jobs in a workflow. GitHub provides hosted runners for Ubuntu Linux, Windows, and macOS, and it also supports integration with self-hosted runners for private operations. GitHub-hosted runners are maintained by GitHub, so their environments cannot be customized. Organizations needing a custom build environment can provision and maintain self-hosted runners. Both GitHub-hosted and self-hosted runners can be used with private or public repositories.

![](https://cdn.wallleap.cn/img/pic/illustration/202408300956787.png?imageSlim)


### 4. GitHub Actions jobs

A job in GitHub Actions consists of a series of steps executed on the same runner. Jobs can run either in parallel or sequentially, depending on the dependencies defined in the workflow. Jobs that do not rely on the output of other jobs can run in parallel, which helps reduce the overall build time.

### 5. GitHub Actions steps

GitHub Actions steps are the individual tasks or commands that make up a job. Each job consists of one or more steps that are executed sequentially. Steps can either run scripts or actions — the reusable code packages. The script allows you to run a series of shell or bash commands in the runner’s environment. Tasks like installing dependencies, running build commands, and testing are performed using scripts. 

### 6. Actions

Actions in GtHub Action are reusable code packages used as steps in a workflow. They perform tasks like setting up environments, running tests, deploying code, etc., which help automate various steps involved in the software development lifecycle.

You can use actions shared by the community/GitHub or create custom ones. GitHub Actions offers three types of custom actions: JavaScript actions, which run directly on the runner machine; Docker container actions, which run in a Docker environment; and composite actions, which combine multiple workflow steps into a single action.

### 7. GitHub Actions artifacts

GitHub Actions artifacts are files or directories that are uploaded and stored after a job in a GitHub Actions workflow is completed. These artifacts enable you to retain and access the job’s output for further processing within the pipeline or by other services. Examples of artifacts include versioned executables, logs, and test results.

### 8. GitHub Actions secrets

Build and deploy activities often require access to sensitive information such as API keys, tokens, and passwords. GitHub Actions provides a secure method to configure and store this sensitive information as secrets. These secrets are stored in encrypted form as environment variables, which are made available to the runners during each workflow run. Thus, workflows can access and use the necessary sensitive information securely.

### What is the difference between a GitHub action and a workflow?

A GitHub “Action” and “Workflow” are both key components of the GitHub Actions platform. A workflow is an automated process triggered by specific events like push, pull, or schedule, defined in YAML files in the .github/workflows directory. It consists of one or more jobs, each containing steps. Actions are reusable units of code within these workflows, performing individual tasks.

While workflows orchestrate automation, actions provide the building blocks, enabling task reuse across different workflows. Custom actions can be created or used from the GitHub Marketplace, facilitating efficient and consistent task execution in software development processes.

💡 You might also like:

- [Should you manage your IaC with GitHub Actions?](https://spacelift.io/blog/infrastructure-as-code-with-github-actions)
- [How to Manage & Scale Terraform with GitHub Actions](https://spacelift.io/blog/github-actions-terraform)
- [Top 10 Most Popular Jenkins Alternatives](https://spacelift.io/blog/jenkins-alternatives)

## Tutorial: How to create a GitHub Actions workflow[](https://spacelift.io/blog/github-actions-tutorial#tutorial-how-to-create-a-github-actions-workflow)

Follow these steps to create GitHub Actions workflow:

1. Write the application code.
2. Create a YAML file to define the actions.
3. Configure a build job.
4. Test your GitHub Action workflow.
5. Configure secrets for GitHub Actions.
6. Configure upload to S3 step.
7. Define the deployment job and access the artifact.
8. Deploy to EC2.

### Continuous integration with GitHub Actions

Continuous Integration (CI) is the practice of automating the build and test jobs and providing early feedback before integrating the new code changes into the central repository. This ensures the stability of the software being deployed/delivered. With GitHub Actions, you configure the sequence of jobs in a workflow to create an end-to-end build process. 

Note that the activities involved in the build phase precede deployment.

#### Step 1: Write the application code

The example below implements a CI pipeline for a basic application written in Go. This is a simple math server that exposes a math addition operation. The API takes in a couple of integers as query parameters and responds with the sum. The code below also hands a couple of String to Int conversion scenarios.

```go
package main

import (
	"fmt"
	"net/http"
	"strconv"
)

func Add(a, b int) int {
	return a + b
}

func additionHandler(w http.ResponseWriter, r *http.Request) {
	values := r.URL.Query()

	aStr := values.Get("a")
	bStr := values.Get("b")

	a, err := strconv.Atoi(aStr)
	if err != nil {
		http.Error(w, "Parameter 'a' must be an integer", http.StatusBadRequest)
		return
	}

	b, err := strconv.Atoi(bStr)
	if err != nil {
		http.Error(w, "Parameter 'b' must be an integer", http.StatusBadRequest)
		return
	}

	result := Add(a, b)

	fmt.Fprintf(w, "%d", result)
}

func main() {
	http.HandleFunc("/addition", additionHandler)

	fmt.Println("Server listening on port 8080....")
	http.ListenAndServe(":8080", nil)
}
```

From the terminal, navigate to the root of this project and run ‘go run .’ to run the application. If it displays the message `Server listening on port 8080….`, it means it has run successfully. Test the simple math API exposed by this application by accessing the localhost, as shown below.

![](https://spacelift.io/_next/image?url=https%3A%2F%2Fspaceliftio.wpcomstaging.com%2Fwp-content%2Fuploads%2F2024%2F07%2F7-github-actions-deployment.png&w=3840&q=75)

This is simple application code managed in a GitHub repository. In the next steps, we will begin writing the GitHub Actions workflow to automatically build, test, deliver, and deploy the code.

#### Step 2: Create a YAML file to define the actions

In the project’s root directory, create a subdirectory, `.github/workflows`. Within this directory, create a file with the .yml extension. In this example, we have named it `go.yml`. This .yml file is automatically interpreted by GitHub Actions as a workflow file. Here, we begin by configuring the triggers. 

The YAML code below is placed at the very beginning of the go.yml file.

```yaml
name: Go

on:
  push:
    branches: [ "main" ]
```

The name parameter just names this workflow as `Go`. `on: push: branches:` part of the workflow file mentions the list of branches for which the workflow should be triggered automatically. In this case, we want the workflow to trigger automatically for the ‘main’ branch.

#### Step 3: Configure a build job

Large applications written in the Go programming language require a build step because it is a statically typed, compiled language. This generates a binary, which is then deployed to the servers. The first job, which we define in the workflow, is to build this binary. 

As mentioned earlier, each job runs on a fresh instance of a runner. Think of this as a fresh virtual machine where the required dependencies are not available. 

The code below defines all the steps required to build a Go application:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Set up Go
      uses: actions/setup-go@v4
      with:
        go-version: '1.21.5'

    - name: Build
      run: go build -v ./...
```

First, we defined a job named `build`. The job name is not part of the syntax, and it could be anything that makes sense to you. Next, we need to specify the `type` of runner we want to use to perform this job. GitHub Actions supports various operating systems like Ubuntu, Microsoft Windows, or MacOS. For our purpose, we will use the Ubuntu environment.

The first step in this job is to check out the application’s source code on the runner. This clones the files in the GitHub repository on the runner VM. In the next step, we set up the Go compiler with the desired version. Finally, in the third step, we run the go build command. The binary thus generated is made available on the runner locally.

Thus, in this step, we have just built the binary for the Go application. If you now push any changes to the source code repository on the main branch, this workflow will automatically trigger and follow these steps each time. 

However, note that because these runners are only made available to perform the jobs, they are revoked when all the jobs are done. This also means that the binary thus generated is also lost after a workflow run.

#### Step 4: Test your GitHub Action workflow

For the next step, we want to test the application before we deliver the binary. The purpose of this test is to quickly identify any issues and provide feedback to the developers before the workflow proceeds to the next phase.

As a best practice, the source should always have unit test cases defined, ensuring maximum coverage. This is true for applications written in any programming language. For this example, let us first write the unit test that ensures the logic written to perform simple math operations is correct. Go provides a ‘testing’ package that helps in writing test cases for the application source code. The code below is part of the main_test.go file — the name of the file is driven by the testing framework.

```go
package main

import (
	"io/ioutil"
	"net/http/httptest"
	"strconv"
	"testing"
)

func TestAdditionHandler(t *testing.T) {
	// Test case 1
	req1 := httptest.NewRequest("GET", "/addition?a=3&b=5", nil)
	w1 := httptest.NewRecorder()
	additionHandler(w1, req1)
	resp1 := w1.Result()
	defer resp1.Body.Close()
	body1, _ := ioutil.ReadAll(resp1.Body)
	result1, _ := strconv.Atoi(string(body1))
	expected1 := 8
	if result1 != expected1 {
		t.Errorf("Test case 1 failed, expected %d but got %d", expected1, result1)
	}
}
```

The test case above makes an API call to the /addition API with specific inputs and expects the corresponding output. If the simple math operation logic is flawed, it fails the test case, and the GitHub Action workflow also fails and does not proceed.

To run this test in the GitHub Actions workflow, go back to the go.yml file and add the following step in the build job defined above.

```yaml
- name: Test
      run: go test -v ./...
```

Push the code to the main branch and observe the pipeline run. As seen from the screenshot below, the test cases are run during the ‘Test’ step and their results are also printed in the logs.

![](https://spacelift.io/_next/image?url=https%3A%2F%2Fspaceliftio.wpcomstaging.com%2Fwp-content%2Fuploads%2F2024%2F07%2F4-build-github-actions-workflow.png&w=3840&q=75)

Any changes to the source code that follow should now be tested using this same step. It is the developer’s duty to make changes to the source code or add more code to make corresponding changes to the unit tests as well.

If the tests pass, we have successfully implemented Continuous Integration (CI) for our simple math server.

### Continuous delivery with GitHub Actions

Positive test results mean that we can now safely proceed to deliver/deploy the application to its target destination. Delivery is different from deployment. In continuous delivery, we build and deliver the artifacts, and a separate process is then responsible for the actual deployment of the artifacts on target servers. In continuous deployment, the artifacts are actually deployed on the servers.

In the example discussed so far, the binary generated is the artifact to be delivered. Since the binary is lost when runners phase out after workflow completion, it makes sense to upload a binary to persistent storage before the runner is released.

#### Step 5: Configure secrets for GitHub Actions

In this example, we are using S3 buckets to store the artifacts, so we need to configure the permissions for the runner VM that allow it to perform ‘PUT’ operations on the target bucket. Accessing AWS using the CLI requires credentials like Access Key and Secret Access Key.

Optionally, we would also need the bucket name after configuring the S3 bucket to be stored as a secret variable and for reuse purposes.

Navigate to your GitHub repository > Settings > Secrets and variables > Actions, and configure these secrets as shown below.

![](https://spacelift.io/_next/image?url=https%3A%2F%2Fspaceliftio.wpcomstaging.com%2Fwp-content%2Fuploads%2F2024%2F07%2F5-github-actions-secrets.png&w=3840&q=75)

**Note:** For now, ignore the `SECRET_KEY`; we will return to it in the upcoming sections.

Also, note the names of all the secrets, as we will need them to configure further steps in our Github Actions workflow.

#### Step 6: Configure upload to S3 step

Back in the go.yml file, configure a new step to upload the artifact (binary) to the destination S3 bucket. The configuration is as below.

```yaml
- name: Upload Binary to S3
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.ACCESS_KEY }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.SECRET_KEY }}
        AWS_DEFAULT_REGION: "eu-central-1"
      run: |
        aws s3 cp ./simplemath s3://${{ secrets.TARGET_BUCKET }}/
```

I have named this step `Upload Binary to S3`. Choose a name that makes sense to you. First, we set the environment variables for the runner where we are setting the AWS credentials. The values for these are being retrieved from the secrets we configured in the previous step. Note how the secrets are accessed using `secrets.` keyword, wrapped in double curly brackets, preceded by a $ sign.

In the run part of this step, we [use AWS CLI to run the `s3 cp` command](https://spacelift.io/blog/aws-s3-cp). In this example, the binary generated is named `simplemath` because that is the name I chose for this Go module while developing. It could be different in your case. To be sure, check the first line of the `go.mod` file.

Note that, like the Go compiler, we were not required to install AWS CLI explicitly. GitHub Actions runners come equipped with it and other utilities. If in doubt, check the [documentation](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners#preinstalled-software) for each runner.

When the pipeline is run, it builds the binary, which the upload step (this step) uploads to the target S3 bucket defined in the secret variable. The screenshot below shows the uploaded binary as well as the name of the bucket used for this example.

![](https://spacelift.io/_next/image?url=https%3A%2F%2Fspaceliftio.wpcomstaging.com%2Fwp-content%2Fuploads%2F2024%2F07%2F6-github-actions-artifacts.png&w=3840&q=75)

In this step, we have successfully delivered the software to its target destination.

### Continuous deployment with GitHub Actions

GitHub Actions help deploy the application on servers or Kubernetes clusters. Typically, this is the final phase of the software development lifecycle and also means the end of an iteration. We’ll explore the continuous deployment aspect of GitHub Actions by deploying the simple math server on an EC2 instance. 

As a prerequisite, configure an EC2 instance with appropriate access enabled for the deployment. The EC2 instance thus created should use a key pair for logging in. The private key of this key pair is stored as the `PRIVATE_KEY` secret variable, as seen in the screenshot in Step 4. This is required for the next steps to work. 

We will also create a new job in the same workflow to keep CI and CD steps separate.

#### Step 7: Define the deployment job and access the artifact

Append the go.yml file with the code below. Here, we create a new job named `deploy`. The first argument, `needs`, specifies the dependency on the first job, `build`. If we do not specify this, both the build and deploy jobs will run in parallel. This will end up deploying the wrong file with older code to the EC2 instance or failing if there is no file present in the S3 bucket.

```yaml
deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
    - name: Download S3
      uses: joutvhu/download-s3@v1.0.0
      with:
        aws_access_key_id: ${{ secrets.ACCESS_KEY }}
        aws_secret_access_key: ${{ secrets.SECRET_KEY }}
        aws_region: "eu-central-1"
        aws_bucket: ${{ secrets.TARGET_BUCKET }}
        target: .
```

As mentioned earlier, each job runs on a fresh runner VM. Thus, we need to select the appropriate OS again by specifying the `runs-on` argument. For similar reasons, this fresh runner VM also has no access to the Go binary. Thus, we have to access it from the same S3 bucket where we stored it in the `build` phase.

The first step in the `deploy` job uses the secrets configured in Step 4 to access and download the binary on the runner VM, placing the binary in the home directory. Knowing the download location is important because we will use it to upload and deploy the binary to the target EC2 instance in the next step.

Note that, in this step, we have made use of the existing GitHub Action published by `joutvhu` from the community. This saves the effort of writing the download logic from scratch.

#### Step 8: Deploy to EC2

In this step, we will also leverage the package published by `cross-the-world` to perform the deployment. Under the hood, in this step the runner: 

1. Uses the private key to log into the EC2 VM
2. Copies the simple math binary downloaded from the S3 bucket to the EC2 user’s home directory
3. Adds execution permission to this binary
4. Runs the binary

```yaml
- name: ssh-scp-ssh-pipelines
      uses: cross-the-world/ssh-scp-ssh-pipelines@v1.1.4
      with:
        host: 3.70.96.177
        user: ec2-user
        key: ${{ secrets.PRIVATE_KEY }}
        scp: ./simplemath => /home/ec2-user/app/simplemath
        last_ssh: chmod +x /home/ec2-user/app/simplemath/simplemath
```

If everything has gone well so far and the inbound and outbound rules are configured correctly, the /addition API should be accessible from the browser as shown below:

![](https://spacelift.io/_next/image?url=https%3A%2F%2Fspaceliftio.wpcomstaging.com%2Fwp-content%2Fuploads%2F2024%2F07%2F3-github-actions-api.png&w=3840&q=75)

We have successfully deployed the simplemath application on the EC2 instance server.

Remember that the complete workflow will be triggered every time you push or merge changes to the main branch of this repository. Once these actions are set, the software development team can perform multiple iterations and release multiple features quickly and safely.

## GitHub Actions examples[](https://spacelift.io/blog/github-actions-tutorial#github-actions-examples)

We have seen how GitHub Actions provides a way to build automation workflows quickly. As a general rule, always check the available actions on [GitHub Marketplace](https://github.com/marketplace?type=actions&page=1) to avoid repetition and save time. Let’s look at some GitHub Actions Marketplace examples that cover several use cases.

### Example 1: Environment setup

Applications written in various languages require the corresponding compilers, runtimes, environment variables, etc., to be set up before they can be operated on successfully. You can find many Github Actions that help set up such environments on the runner before the actual steps are executed. 

The example below uses “Setup Node.js environment” GitHub Action to install Node.js environment by tweaking any of the relevant attributes below. Ideally, this step should be followed by the npm package installation and testing steps.

[Source](https://github.com/marketplace/actions/setup-node-js-environment)

```yaml
- uses: actions/setup-node@v4
  with:
     node-version: ''
     node-version-file: ''
     check-latest: false
     architecture: ''
     token: ''
     cache: ''
     cache-dependency-path: ''
     registry-url: ''
     scope: ''
     always-auth: ''
```

### Example 2: Docker images

Applications to be deployed in containerized environments require explicit build and publish steps for managing Docker container images. This is required for every push/pull request, as the changes are to be reflected in the new image. Cloud Posse offers the [Docker build and push action](https://github.com/marketplace/actions/docker-build-and-push) to do this. 

In the example below, we have specified the credentials for the Docker repository and other details, like the build platform. This information is enough for this action to automatically build and push the Docker image. 

[Source](https://github.com/marketplace/actions/docker-build-and-push)

```yaml
name: Push into main branch
  on:
    push:
      branches: [ master ]

  jobs:
    context:
      runs-on: ubuntu-latest
      steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Build
        id: build
        uses: cloudposse/github-action-docker-build-push@main
        with:
          registry: registry.hub.docker.com
          organization: "${{ github.event.repository.owner.login }}"
          repository: "${{ github.event.repository.name }}"
          login: "${{ secrets.DOCKERHUB_USERNAME }}"
          password: "${{ secrets.DOCKERHUB_PASSWORD }}"
          platforms: linux/amd64,linux/arm64

    outputs:
      image: ${{ steps.build.outputs.image }}
      tag: ${{ steps.build.outputs.tag }}
```

### Example 3: Security scanning

GitHub Actions also enable the ScOps with their flexible workflow. Various Github Actions are available to perform security scanning of files, source code, and container images. In the example below, we set up source code scanning for Go applications as a step in the GitHub Actions workflow. Once set up, after the checkout step, this action inspects the source code and provides results to be leveraged in the development process.

[Source](https://github.com/marketplace/actions/gosec-security-checker)

```yaml
name: Run Gosec
on:
  push:
    branches:
      - master
  pull_request:
    branches:
      - master
jobs:
  tests:
    runs-on: ubuntu-latest
    env:
      GO111MODULE: on
    steps:
      - name: Checkout Source
        uses: actions/checkout@v3
      - name: Run Gosec Security Scanner
        uses: securego/gosec@master
        with:
          args: ./...
```

## Using Spacelift for CI/CD[](https://spacelift.io/blog/github-actions-tutorial#using-spacelift-for-cicd)

Compared to building a custom and production-grade infrastructure management pipeline with a CI/CD tool like GitHub Actions, adopting a collaborative infrastructure delivery tool like [Spacelift](https://spacelift.io/) feels a bit like cheating. 

Many of the custom tools and features your team would need to build and integrate into a CI/CD pipeline already exist within Spacelift’s ecosystem, making the whole infrastructure delivery journey much easier and smoother. It provides a [flexible and robust workflow](https://docs.spacelift.io/concepts/run/) and a native GitOps experience. It detects configuration drift and reconciles it automatically if desired. Spacelift runners are Docker containers that allow any type of customizability. 

![](https://cdn.wallleap.cn/img/pic/illustration/202408300957820.png?imageSlim)


Security, guardrails, and [policies](https://docs.spacelift.io/concepts/policy/) are vital parts of Spacelift’s offering for governing infrastructure changes and ensuring compliance. Spacelift’s built-in functionality for developing custom modules allows teams to adopt testing early in each module’s development lifecycle. [Trigger Policies](https://docs.spacelift.io/concepts/policy/trigger-policy) can handle dependencies between projects and deployments.

If you want to learn more about Spacelift, [create a free account today](https://spacelift.io/free-trial) or [book a demo with one of our engineers](https://spacelift.io/schedule-demo).

## Key points[](https://spacelift.io/blog/github-actions-tutorial#key-points)

GitHub Actions is a flexible tool for automating software development workflows. Its vast ecosystem of actions and the ability to create custom actions streamline various tasks, from building and testing your code to deploying your applications and managing your infrastructure. Depending on the jobs configured in your GitHub Actions workflow, they ensure code quality and deliver software more efficiently.

In this beginner-friendly GitHub Actions tutorial, we just scratched the surface by covering the basics, including how to create workflows with GitHub Actions, use built-in and third-party actions, and leverage advanced features like environment secrets and artifacts. We also explored practical examples that demonstrated how to set up continuous integration, delivery, and deployment pipelines for different types of projects.
