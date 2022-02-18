# CI/CD For a Springboot Application on AWS ECS 
http://54.242.136.181:8080/orders

![Diagram for presentation](https://user-images.githubusercontent.com/97601366/154591988-0ac89658-90e0-49e0-9753-2c20b716b1c1.png)

This is a CI/CD Model for building and updating docker images from github as a repo and ecs and a target.

This setup does the following:

1. Creates a source(repo) as github where the dockerfile, sourcecode(Javafiles) as well as configurations(buildspec) and uri of image, etc., are stored.
2. Once this source is created, the pipeline can then be connected by linking this repo as source in the first stage of the codepipeline.
3. In the second stage, codebuild takes the dockerfile and javafiles and creates an image with it.
4. the image created is the stored in a private repository in ecr
5. each time there is a commit on github, the pipeline is triggered and codebuild compiles the docker image. 
6. ecr is then configured to use the latest version of the image gotten from ecr.

Critical steps to remember:
Congigure IAM roles for Codebuild
Update Repon URI in buildspec.yml
Expose ports in Security group for ecs to be accessible
Take note of the branch you're commiting to.

http://54.242.136.181:8080/orders
