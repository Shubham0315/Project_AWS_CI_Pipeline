# AWS_CI_Pipeline 

- This project is a use case of AWS Code commit, AWS Code pipeline, AWS Code Build
- Start with code build so that we can build project with all steps like Code checkout, Build and UT, Code scan, Image build, image scan and image push. After that create pipeline for orchestration.

![image](https://github.com/user-attachments/assets/b8255c2d-d0ac-4ccf-b5b4-d74837033646)

- In GitHub create "simple-python-app"
  - This repo has veru basic flask code
 
![image](https://github.com/user-attachments/assets/8f1698ba-4530-468d-b239-1eb1bd546f8a)

  - We also have requirements file for python app (flask)
  - We've dockerfile here to build application image and push it to dockerhub. So we've python related dockerfile as well. Here we're using python based image, created WORKDIR, COPY requirements file, exposed port

![image](https://github.com/user-attachments/assets/fd697807-d8d9-4275-a3a5-f18dd07ee30a)

  - We also have buildspec.yml. We'll write this file in our pipeline.

#1. AWS Code Build

- Go to AWS Code build and create AWS Code build project as below.
- Project Name :- **python-flask-service-app**

![image](https://github.com/user-attachments/assets/9e013452-92d7-462e-a21a-57dcec2f0640)

- We can also restrict no of concurrent builds here

- In source section, provide Code Commit repo as GitHub. If not conneceted to GitHub account, *"Connect with OAuth"* or *"Connect with GitHub personal Access token"*

![image](https://github.com/user-attachments/assets/54d75c29-323c-4bcf-80af-9e25d84be103)

- Now our code build is integrated with GitHub

- If GitHub is connected with AWS, provide repository type as *"Public repo"* or *"Repo in my GitHub account"* and provide URL

![image](https://github.com/user-attachments/assets/53e0cb5f-65df-4d30-a662-b18798584b2f)

- Go to environment section now.
  - As AWS Code pipeline acts as orchestrator which invokes AWS Code build, so we need to provide environment for AWS Code build
  - Here environment will be VM image or Docker image. Environment image will be managed image or custom image

![image](https://github.com/user-attachments/assets/29578973-aa65-402d-8019-930658c2d8ce)

  - In managed images choose image required (Amazon Linux, Ubuntu or Windows). Then choose runtime and choose latest image. Here AWS code build we use as Builds service. As this is managed service, AWS will provide you images where we can run our code build on them. This is provided by AWS only
  - On contrary, in Jenkins what we do is create worker nodes on top of which we install operating systems and run pipelines on the environment. 

- Now select "Service Role". In AWS, for Code build to perform action on AWS needs service role or an IAM role. As it is service performing some action, we're using Role not IAM users. Users dont perform action here.
- Go to IAM Roles to create one like below which uses code build service and grant required permission for roles. We can give any name

![image](https://github.com/user-attachments/assets/d12e8525-5108-4a2e-8604-83e06488fbf7)


- Below is snap of adding role to our Build project

  <img width="635" alt="image" src="https://github.com/Shubham0315/AWS_CI_Pipeline/assets/105341138/e509e721-1819-4a00-98c7-353505173ab6">

Build Spec
- 
- For all stages of pipeline to execute we need to write code which we write here for checkout, image creation, image scanning. We can use Build spec file from github repo or we can write commands specific manually. To write manually "switch to editor"

![image](https://github.com/user-attachments/assets/990e48b3-41e0-4a06-ac69-7ba0c6ba61e0)

- Here write the stages as below.
  - We can start with "Phases" stage. Install phase, we need to select runtime needed which is python here.
 
![image](https://github.com/user-attachments/assets/8eee3c52-f361-40cd-ba3a-232d481dca0c)

  - Start writing pre build stage. Here we can define the requirements.txt file for our image to run as it installs dependencies of flask mentioned (pre-requisite)

![image](https://github.com/user-attachments/assets/1be358e0-89c2-415e-9c65-20c0abafa826)

  - Now go to build stage. We need to checkout the code which will happen automatically as we've already integrated GitHub repo.
  - Next step is to build the app, build docker image and push it. Write required commands to build image. Here as of now we need to build image and push to docker hub
  - So in below snap we can see, we navigated to the directory (cd .), started build, executed docker commands to create image from dockerfile and push the same to Docker hub.
  - To build docker image, we need to pass image name, user and password which we'll store in AWS syetems manager later. If we directly put password and username here, whoever has access to this buildspec file can use it (security)

  <img width="378" alt="image" src="https://github.com/Shubham0315/AWS_CI_Pipeline/assets/105341138/66619572-2767-4dd6-8869-cc2429157740">
  
  - We've given tag in above image as we wont update them as they are sensitive. We can store them in AWS credentials manager. If we provide user and password in tag in above snap, anyone who opens the file can use that compromising security.

- As we're only doing CI here, in post-build actions add only success statement. So our final buildspec.yml file will look like :-

![image](https://github.com/user-attachments/assets/f8f8d851-0bca-45db-93fb-47e7a0db7e16)

- After all is done, create project of build.

![image](https://github.com/user-attachments/assets/82089f85-0787-4aea-bf99-93e1da658851)

- To create role, go to IAM role - create role - add permissions - provide name of role

![image](https://github.com/user-attachments/assets/6a9b36f4-ef0e-4576-8c82-7cfefa8ac70f)


- If now we trigger the build, it will take code from Code Commit and will trigger build.


# Store Sensitive Information

- Go to "AWS System Manager" and go to "Parameters Store"
- While naming follow standard like project name/parameter so even if we store parameters for 100s of services, we can easily identify which parameter us for what use case.
  Select "Secure string". Then put value of parameter inside "Value" attribute
  /myapp/docker-credentials/username  :- Shubham315
  /myapp/docker-credentials/password  :- Charger@7701
  /myapp/docker-credentials/docker_registry_url       :- docker.io  (as credentials are already provided)

  <img width="958" alt="image" src="https://github.com/Shubham0315/AWS_CI_Pipeline/assets/105341138/b88ddc1c-4396-425c-a166-8bf537f7cc36">

# Configure Credentials in buildspec file

- Go to Code Build. Open Build Details and edit buildspec.yml and pass the credentials as below

  <img width="495" alt="image" src="https://github.com/Shubham0315/AWS_CI_Pipeline/assets/105341138/d35dbd86-f1aa-45ef-8a77-3d260637125b">

- The build and push command also need to be updated as per below.

  <img width="790" alt="image" src="https://github.com/Shubham0315/AWS_CI_Pipeline/assets/105341138/f68b5c70-0952-4d3a-9908-c880b9db5a6d">

# Error Rectification

- Now start the Build for first time. It will fail at first if we have done manual errors.

1. First Failed attempt due to invalid phase names :- pre_build and post_build
   <img width="794" alt="image" src="https://github.com/Shubham0315/AWS_CI_Pipeline/assets/105341138/bd4548f0-d311-47f5-a7d6-e966cb3c2c66">

2. Second error is due to permissions
- Here provided the docker credentials but did not grant AWS code build access to system manager.
- We have attached role but that role doesn't have permission to talk with AWS System Manager
- Go to IAM, our role and attach policies of SSM full access (SSM is System Manager)

<img width="949" alt="image" src="https://github.com/Shubham0315/AWS_CI_Pipeline/assets/105341138/28cc9133-0ce7-4f33-8bc9-3830184d4122">

3. Now we get 3rd error due to misconfiguration

<img width="761" alt="image" src="https://github.com/Shubham0315/AWS_CI_Pipeline/assets/105341138/a85356b9-7c28-44df-ad32-285f4435b38e">

- We need to provide repository name in lowecase while configuring Build project as well as in System manager parameter. Here my parameter name was in uppercase which is not acceptable.


4. Now we get below error while build failed 4th time
- Push command should not contain dot (.)

<img width="652" alt="image" src="https://github.com/Shubham0315/AWS_CI_Pipeline/assets/105341138/128078ad-bd35-492b-9e5f-d48cecd46d67">

5. Below error might be due to our credentials are not proper

<img width="750" alt="image" src="https://github.com/Shubham0315/AWS_CI_Pipeline/assets/105341138/73e7033f-e898-4fb3-8b90-fa6160a52f5c">

- By default AWSCode build doesnt allow to create docker images on build system it provides. Click on override image option and enable "Privilege"

<img width="644" alt="image" src="https://github.com/Shubham0315/AWS_CI_Pipeline/assets/105341138/c403edf5-3ffb-40a7-b847-fec0e3b7d3bd">

- Here we missed the docker login command

- Also in push command, it should be docker.io not the URL

<img width="546" alt="image" src="https://github.com/Shubham0315/AWS_CI_Pipeline/assets/105341138/1927a0a9-b038-4113-a9c7-75489ae49ad5">

- Finally we got our CI pipeline implemented successfully.

<img width="610" alt="image" src="https://github.com/Shubham0315/AWS_CI_Pipeline/assets/105341138/348fc783-7708-43e5-a120-e5397f7248bf">

- After resolving all errros, we have successuflly pushed our image to docker hub

<img width="959" alt="image" src="https://github.com/Shubham0315/AWS_CI_Pipeline/assets/105341138/938e2c8d-5d7d-421c-b112-fb94ae039a22">





#2 AWS Code Pipeline

- Till now, we are triggering build manually. Pipeline has to be triggered automatically which is to be doen by Code Pipeline when change is oushed to github repository.

- Go to Code Pipeline and create as "python-app". Connect to GitHub (Version2)
- This connection is established as when code commit is done, it will auto trigger pipeline for the build.

<img width="744" alt="image" src="https://github.com/Shubham0315/AWS_CI_Pipeline/assets/105341138/51af7519-e6be-4dae-bf49-abef8421ce74">

- Set build provider and project name as below

  <img width="959" alt="image" src="https://github.com/Shubham0315/AWS_CI_Pipeline/assets/105341138/5c0ff32e-b6d8-4715-b879-ec4e59f12feb">

  - By this way our CI Pipeline is created !!! If we commit any change in repository, pipeline gets triggered automatically.






  

    




- 



