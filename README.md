![154591988-0ac89658-90e0-49e0-9753-2c20b716b1c1](https://user-images.githubusercontent.com/97601366/155172472-c02083b0-b956-4013-bacb-5cbfea62bf35.png)

## CI/CD For a Springboot Application on AWS ECS 

This is a CI/CD Model for building and updating docker images from github (as a repo) and ECS cluster as a target.

## Documentation

This project assumes you are have some basic knowledge of Springboot applications, Docker, Git, Github and AWS Pipeline.
If you are not familiar with these concepts/tools, please visit the following links:
* https://www.docker.com/101-tutorial
* https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#getting-started
* https://docs.aws.amazon.com/whitepapers/latest/introduction-devops-aws/welcome.html
* https://docs.aws.amazon.com/codebuild/latest/APIReference/codebuild-api.pdf
* https://docs.github.com/en/get-started/quickstart/create-a-repo

## Summary:
* Creates a source(repo) as github where the dockerfile, sourcecode(Javafiles) as well as configurations(buildspec) and uri of image, etc., are stored.
* Once this source is created, the pipeline can then be connected by linking this repo as source in the first stage of the codepipeline.
* In the second stage, codebuild takes the dockerfile and javafiles and creates an image with it.
* The image created is the stored in a private repository in ECR.
* Each time there is a commit on github, the pipeline is triggered and codebuild compiles the Docker image.
* ECR is then configured to use the latest version of the image gotten from ECR.

## Step 1: Creating a repo as source

Organize the sourcecode for your Springboot application locally(organised properly in directories).
Using git, the following commands will initialize your repo, add a readme, make your first commit and main branch, initialize changes made to your repo locally and push the local version to perform a full sync.

```git init 
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin yourrepo
git push -u origin main
```
For more information on this please visit
https://docs.github.com/en/get-started/using-git/about-git#github-and-the-command-line

## Step 2: Create ECR and copy URI

![image](https://user-images.githubusercontent.com/97601366/155216588-0e458725-95ab-47f3-9cc6-7227b4635fca.png)
![image](https://user-images.githubusercontent.com/97601366/155216789-5ab620de-3b89-4f4d-95c5-61e8e024f525.png)

## Step 3: Specify your buildspec as seen below

```phases:
  pre_build:
    commands:
      - mvn clean install
      - echo Logging in to Amazon ECR...
      - aws --version
      - $(aws ecr get-login --region $AWS_DEFAULT_REGION --no-include-email)
      - REPOSITORY_URI= Repository URI e.g. 484927367294.dkr.ecr.us-east-1.amazonaws.com/javadockerized 
      - COMMIT_HASH=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
      - IMAGE_TAG=build-$(echo $CODEBUILD_BUILD_ID | awk -F":" '{print $2}')
  build:
    commands:
      - echo Build started on `date`
      - echo Building the Docker image...
      - docker build -t $REPOSITORY_URI:latest .
      - docker tag $REPOSITORY_URI:latest $REPOSITORY_URI:$IMAGE_TAG
  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing the Docker images...
      - docker push $REPOSITORY_URI:latest
      - docker push $REPOSITORY_URI:$IMAGE_TAG
      - echo Writing image definitions file...
      - printf '[{"name":"order-service","imageUri":"%s"}]' $REPOSITORY_URI:$IMAGE_TAG > imagedefinitions.json
      - cat imagedefinitions.json
artifacts:
  files:
   - imagedefinitions.json
   - target/order-service.jar
   ```



## Step 4: Create codebuild 
Configure codebuild as follows:

![codebuildeditted](https://user-images.githubusercontent.com/97601366/155218149-32cbe413-c55a-49e5-8e81-bafa1fa1f1a8.png)
![codebuild2eddited](https://user-images.githubusercontent.com/97601366/155219066-d3315295-9111-498f-888b-c362f38aae76.png)
![image](https://user-images.githubusercontent.com/97601366/155219234-516f173c-93f1-432f-bc9e-10a15fe86fd0.png)

While configuring your codebuild, set Github as your source so your Buildspec can be fetched from your repo(source) and ensure that the "priviledged" checkbox is ticked. This would give elevated permissions needed for docker operations.
Trigger codebuild to build initial image.


## Step 5: Create codepipeline, specify the stages(source, codebuild- created earlier)


## Step 6: Create Task Definition and select the container image you created in ECR.

## Step 7:
Create an ECS Cluster, then create a task using the task definition created earlier
Ensure that auto assign IP is enabled and ensure that the security group has its ports exposed(8080 and 80)


## Step 8: Specify your buildspec as seen below


phases:
  pre_build:
    commands:
      - mvn clean install
      - echo Logging in to Amazon ECR...
      - aws --version
      - $(aws ecr get-login --region $AWS_DEFAULT_REGION --no-include-email)
      - REPOSITORY_URI=484927367294.dkr.ecr.us-east-1.amazonaws.com/javadockerized
      - COMMIT_HASH=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
      - IMAGE_TAG=build-$(echo $CODEBUILD_BUILD_ID | awk -F":" '{print $2}')
  build:
    commands:
      - echo Build started on `date`
      - echo Building the Docker image...
      - docker build -t $REPOSITORY_URI:latest .
      - docker tag $REPOSITORY_URI:latest $REPOSITORY_URI:$IMAGE_TAG
  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing the Docker images...
      - docker push $REPOSITORY_URI:latest
      - docker push $REPOSITORY_URI:$IMAGE_TAG
      - echo Writing image definitions file...
      - printf '[{"name":"order-service","imageUri":"%s"}]' $REPOSITORY_URI:$IMAGE_TAG > imagedefinitions.json
      - cat imagedefinitions.json
artifacts:
  files:
   - imagedefinitions.json
   - target/order-service.jar



Obtain the IP address of the task running alongside its ports and you can access the service.

Each time the source(master branch) is modified, the pipeline is triggered and the newer version is updated



