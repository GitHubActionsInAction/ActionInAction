# Hands-on: a docker container action in action

In this hands-on lab you will create a docker container action that uses input and output parameters. Furthermore, you will create a CI build that tests the action every time y change is made to one of the files.

The lab consists of the following parts:

1. [Use the template to create a new repo](#use-the-template-to-create-a-new-repo)
2. [Create the dockerfile for the action](#create-the-dockerfile-for-the-action)
3. [Create the action.yml file](#create-the-actionyml-file)
4. [Create the entrypoint.sh file](#create-the-entrypointsh-file)
5. [Create a workflow to test the action](#create-a-workflow-to-test-the-action)

## Use the template to create a new repo

In this repository, under [Code](/), click on 'Use this template' and select [Create new repository](../../generate).

<img width="350" alt="CH 04_001" src="https://github.com/GitHubActionsInAction/ActionInAction/assets/5276337/b477028d-f939-454a-9f9b-1b326261e06f">

Pick your GitHub username as the owner and enter `MyActionInAction` as the repository name. Make the repository public and click `Create repository from template`.

<img width="600" alt="CH 04_002" src="https://github.com/GitHubActionsInAction/ActionInAction/assets/5276337/3f1a0b86-8b4d-4523-a2bd-d0e19054caac">


## Create the dockerfile for the action

Create a [new file](../../new/main?filename=Dockerfile) called `Dockerfile``. Add the following content:

```Docker
# Container image that runs your code
FROM alpine:latest

# Copies your code file from your action repository to the filesystem path '/' of the container
COPY entrypoint.sh /entrypoint.sh

# Make the script executable
RUN chmod +x entrypoint.sh

# Code file to execute when the docker container starts up ('entrypoint.sh')
ENTRYPOINT ["/entrypoint.sh"]
```

The dockerfile defines the docker container for the action. You could also use an existing image, but we want to build everything from scratch. We use the latest alpine image and just copy a simple shell script into the container.

Commit the file.

## Create the action.yml file

Create a [new file](../../new/main?filename=action.yml) called `action.yml`. Add the following content and replace the placeholder `{GitHub username}` with your GitHub user name:

```YAML
name: "{GitHub username}'s Action in Action"
description: 'Greets someone and returns always 42.'
inputs:
  who-to-greet:  # id of input
    description: 'Who to greet'
    required: true
    default: 'World'
outputs:
  answer: # id of output
    description: 'The answer to everything (always 42)'
runs:
  using: 'docker'
  image: 'Dockerfile'
  args:
    - ${{ inputs.who-to-greet }}
```

This action file defines the action and the input and output parameters. The `runs` section is the part that defines the action type - in this case we use `docker` together with a `Dockerfile` instead of an image.

Commit the file.

## Create the entrypoint.sh file

Create a [new file](../../new/main?filename=entrypoint.sh) called `entrypoint.sh`. Add the following content:

```Bash
#!/bin/sh -l

echo "Hello $1"
echo "answer=42" >> $GITHUB_OUTPUT
```

The script just writes `Hello` and the input parameter `who-to-greet` to the console and sets the output parameter `answer` to 42.

## Create a workflow to test the action

The action is now ready to be used. To see it in action, we'll create a workflow that uses it. Create a [new file](../../new/main?filename=.github/workflows/test-action.yml) called `.github/workflows/test-action.yml`. Add the following content:

```YAML
name: Test Action
on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo to use the action locally
        uses: actions/checkout@v3.5.3
        
      - name: Run my own container action
        id: action
        uses: ./
        with:
          who-to-greet: '@wulfland'
      
      - name: Output the answer
        run: echo "The answer is ${{ steps.action.outputs.answer }}"
      
      - name: Test the container
        if: ${{ steps.action.outputs.answer != 42 }}
        run: |
          echo "::error file=entrypoint.sh,line=4,title=Error in container::The answer was not expected"
          exit 1
```

The workflow will automatically run and execute the action. Inspect the output. You can see that the container was created and outputs th egreeting to the workflow log. The otput parameter can be access in subsequent steps.

<img width="350" alt="CH 04_003" src="https://github.com/GitHubActionsInAction/ActionInAction/assets/5276337/e6d2cbc6-186b-4093-9f63-d6bc31bace75">
