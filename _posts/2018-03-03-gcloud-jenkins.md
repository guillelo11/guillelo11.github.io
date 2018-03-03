---
layout: post
title: 'Google Container Registry and Jenkins'
categories: Docker
excerpt: 'I recently had to deal with the task of experimenting with automating the build and push of the Docker images, using Jenkins, to my company's Google Cloud based container registry.'
---

I recently had to deal with the task of experimenting with automating the build and push of the Docker images, using Jenkins, to my company's Google Cloud based container registry.

After hours of googling and reading plenty of articles here is the base solution I decided to apply.

The first thing I needed was make sure I had the following Jenkins' plugins:
* Docker pipeline
* [Google Container Registry Auth Plugin](https://wiki.jenkins.io/display/JENKINS/Google+Container+Registry+Auth+Plugin)
* [Google OAuth Plugin](https://wiki.jenkins.io/display/JENKINS/Google+OAuth+Plugin)

After that, I set up the Google Cloud credentials for the user we created to use with Jenkins. Then I created a service account key in the credentials page (API Manager > Credentials) in the Google Cloud console. It will download you a json file with the crendetials.

The next step I did was adding the credentials I just downloaded to our Jenkins container. I opened the form to add the credentials in Jenkins (Credentials > Global Credentials > Add Credentials), select 'Google Service Account from private key' as the crendentials' kind, gave them a name and upload the json file.

After those three steps it's now time to write the Jenkinsfile for our proyect with the help of the Docker Pipeline plugin.

The resulting Jenkinsfile was something like the following:

```groovy
#!groovy 

def myImage 

pipeline { 
    agent any 
    stages { 

        stage('Checkout') {
            //Get the code from the repository
            ...
        }

        stage('Build project') {
            //Building the project here
            ...
        }

        stage('Create image') { 

            steps { 
                echo 'Creating image...' 
                script { 
                    myImage = docker.build('eu.gcr.io/foo-1234/bar') 
                } 
                echo 'Image created'
            } 
        } 

        stage('Push to registry') { 
            steps { 
                echo 'Pushing to registry...' 
                script { 
                    docker.withRegistry('https://eu.gcr.io', 'gcr:our-gc-credentials') { 
                        myImage.push('latest')
                    } 
                } 
                echo 'Image pushed' 
            } 
        } 
    } 
} 
```

Note how the credentials are referenced with the **gcr:** prefix and the image's name using the following notation: **gc-proyect-id**/**image-name**.