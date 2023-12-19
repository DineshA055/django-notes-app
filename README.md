


                          JENKINS-DECLARATIVE-CICD-PIPELINE 🚡




![Uploading image.png…]()

1. Pre-request
     package has to up to date 
     Installed JRE-11 java for jenkins runtime
     Installed JENKINS in instance and enabled

2. Check the code is ready in repository for your project note-app code is avaialble

3. Check Docker file is available to build the image

     Starting process:-

1. Instance is up-to-date
2. JRE-11 installed
3. Jenkins logged-In in web-browser through instance ip:8080 port has to enabled in security group
4. Docker has been installed
5. user has been added to docker-group
              If shows error demon is not running please restart or exit and connect the ssh remote instance once
6. Clone you github repo current project directory in to terminal


Now start jenkins to created CI-CD process:-

Step1 :- Create a Pipeline project with name of your project

Step2 :- Copy and paste you current project Github URL in jenkins repository

step3:- In pipeline (choose pipeline script ) 
            Write a pipeline script 
Now am creating dry run only creating steps in pipeline to run

![image](https://github.com/DineshA055/django-notes-app/assets/101075223/3bb01f92-e5f7-4615-befa-b6e9f06bc88c)
![image](https://github.com/DineshA055/django-notes-app/assets/101075223/35f4fbcb-aa5a-4b8d-90fc-446f8d63d753)

      
Now save   ----------> click build-Now

Success on stage build with dry run 

![image](https://github.com/DineshA055/django-notes-app/assets/101075223/a5827f89-441b-404c-8d72-932d8b8b702e)

Console Output :-

![image](https://github.com/DineshA055/django-notes-app/assets/101075223/111d6f1b-00ab-4c6b-a13b-356ac638f992)

Dry run completed 


Now add some steps 
![image](https://github.com/DineshA055/django-notes-app/assets/101075223/b291cf10-3a53-40b7-8551-14f5cf6166f0)

Added steps for Git clone and build the image 

In console o/p we can see error git was cloned bu access denied for building image because 

We need to add jenkins user in docker group

make you can access the jenkins workspace in terminal 

if ok then CMD :- sudo chmod -aG docker jenkins

Some time it will not effect try to re-connect or sudo reboot 

After restrat run the build again   

Stage View shows – Success upt o Build the image

![image](https://github.com/DineshA055/django-notes-app/assets/101075223/0052be44-6ae8-4ff7-971d-a5f5ce0bf3ad)

Up next givew push image sh shell command to push to docker hub

you will get error after given of cmd push image to docker hub

Important we have to provide docker user name and password in Jenkins -> manage jenkin                          --> Security -----> Credentials -----> to store dockerhub username and password safe and encrypted

By calling using environmental varaibles

Before login to docker in terminal -----> if not manually do this

in Pipeline script provide in stage push to dockerhub
      sh 



Add credential for Docker hub 
![image](https://github.com/DineshA055/django-notes-app/assets/101075223/fa3cbdf2-0aa7-4805-8519-3d8fe7c42bcf)

Next write pipeline script 

adding stage in docker hub

withCredentials      -> groovy syntax also it uses ID to acces the username and pass


withCredentials()      --> () it is function 
withCredentials([])   --> ([])passing the list  and inside list have more functions
withCredentials([usernamePassword()])   --> ([usernamePassword()])  # function which search for ID will extract the user name and password 
withCredentials([usernamePassword(credentialsID:"dockerHub",passwordVariable:"dockerHubPass",usernameVariable:"dockerHubUser")])     

-> remainng variable to get username and pasword retrive securley 

Next tag the image to push to docker hub

sh “docker tag note-app $(env.dockerHubUser}/note-app:latest”           #  env using dockerhub user name and passing the value to login next to docker hub

sh "docker login -u  ${env.dockerHubuser} -p${env.dockerHubpass}"


After 28 attempt of build faliure now its working 

And build is success

![image](https://github.com/DineshA055/django-notes-app/assets/101075223/c85e0bb1-9a83-466b-821c-75ff0bbccb4d)


issue faced with resolving on credentails of docker hub and access using variable to call

pipeline {
    agent any 
    
    stages{
        stage("Clone Code"){
            steps {
                echo "Cloning the code"
                git url:"https://github.com/DineshA055/django-notes-app.git", branch: "main"
            }
        }
        stage("Build"){
            steps {
                echo "Building the image"
                sh "docker build -t note-app ."
            }
        }
        stage("Push to Docker Hub"){
            steps {
                echo "Pushing the image to docker hub"
                withCredentials([usernamePassword(credentialsId:"dockerHub",passwordVariable:"dockerHubPass",usernameVariable:"dockerHubUser")]){
                sh "docker tag note-app ${env.dockerHubUser}/note-app:latest"
                sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                sh "docker push ${env.dockerHubUser}/note-app:latest"
                }
            }
        }
        stage("Deploy"){
            steps {
                echo "Deploying the container"
                sh "docker run -d -p 8000:8000 docker_hub_user_name/note-app:latest"
                
            }
        }
    }
}



Now done with docker run we can use one file docker-compose to down and up the application



DOCKER-COMPOSE can be useful to down required application for few minutes and upgrade or changes in code can be up with few minutes using docker-compose

Docker-compose.YAML file manifest   (one file)

In production stage befor updating we will down the container and upadte the change like remove previous image will pull new image in conatiner again it will make up using docker-compose.yaml manifest technique.


Let go do change in deploy stage in groovy syntax:- 


using CMD :- you can check local pc 

to up the container CMD :- 

before install :- sudo apt-get install docker-compose

Next :- look for docker-compose.yaml file ready in github repo

clone it again or update it in terminal

check ls in directory


next :- 

CMD :- docker-compose up -d 

![image](https://github.com/DineshA055/django-notes-app/assets/101075223/536e6585-bb32-4624-b0c5-c3ab1f1ba035)

single cmd can pull the image from docker hub and expose to run the application in given conatiner 


If you want to update or change the image 

CMD :- docker-compose down 

now container is down 

CMD:- you can change the image and update again use cmd to up the container

cmd :- docker-compose up -d




Now you can change the deployment stage to docker down down and up

because already a conatiner is running if we want to use docker-compose

We have to down the container and up again

stage view – success of down the container and container is UP now

Success
![image](https://github.com/DineshA055/django-notes-app/assets/101075223/b3bdfa83-fc59-4a29-ac07-10b7317d4659)



In pipeline i want use an easy way

choose  pipeline script from SCM instead of choosing pipeline script


Now choose pipeline script from SCM :-

choose git repository 

change branch :- */main

path :- Jenkinsfile                        



Now save build now 

THIS IS DECLARATIVE CHECKOUT 

![image](https://github.com/DineshA055/django-notes-app/assets/101075223/95f55399-ed0a-4503-a778-09da66cd19cf)




We use webhook in github when there is pull or push any commit based on how we add checkpoint when we set weebhook in repo setting -> choose webhook provide 
link of jenkins URL +/github-webhook

Now check validated with webhook request if shows warning symbol so connection not ready so conform the security group allowed for 8080

Now you receive response you can see green so connection is fine

Now you can commit any changes jenkins look for changes and build automatically

Below you can see changes 

![image](https://github.com/DineshA055/django-notes-app/assets/101075223/bc3b1501-5565-48d4-ac96-66fbe3e6bed4)

![image](https://github.com/DineshA055/django-notes-app/assets/101075223/684a418c-34f5-4152-9345-7738bb67077d)







Now you see result also show build was success
![image](https://github.com/DineshA055/django-notes-app/assets/101075223/9c2362cb-db62-4e47-bfbf-0902cbbb8fd7)





**************END OF PRACTISE JENKINS DECLARATIVE ************************************************************there Infinity to go *********
