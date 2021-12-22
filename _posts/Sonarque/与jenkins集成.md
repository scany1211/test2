In our recent guide, we took the [installation of Jenkins](https://computingforgeeks.com/how-to-install-jenkins-on-centos-rhel-8/) and [installation of SonarQube](https://computingforgeeks.com/install-sonarqube-code-review-centos/) by the horn and we managed to get them working. After that, we realize  that we need to have Jenkins successfully communicate with SonarQube so  that we can have our code scanned when that stage in our pipeline is  required. In this succinct guide, we are going to tackle this and ensure that our pipeline completes successfully with SonarQube scanning  involved.

## Step 1: Install SonarQube Scanner Plugin

In order for Jenkins to communicate with SonarQube, we need special  plugins to make it happen. Simply login to Jenkins and proceed to  install tools that will allow us to connect and communicate with  SonarQube. Luckily, there is an amazing plugin ready for you to install  and configure. Head over to your Jenkins Server Web portal, click on “***Manage Jenkins\***” > “***Manage Plugins\***” > Click on the “***Available tab\***” then search for SonarQube. The screenshots for the above steps are shared below.

<iframe id="google_ads_iframe_/1254144,21789414456/computingforgeeks_com-box-3_0" title="3rd party ad content" name="google_ads_iframe_/1254144,21789414456/computingforgeeks_com-box-3_0" scrolling="no" marginwidth="0" marginheight="0" style="border: 0px none; vertical-align: bottom; width: 300px; height: 250px;" srcdoc="" data-google-container-id="3" data-load-complete="true" width="300" height="250" frameborder="0"></iframe>



[![manage jenkins](https://computingforgeeks.com/wp-content/uploads/2021/06/manage-jenkins.png?ezimgfmt=ng%3Awebp%2Fngcb23%2Frs%3Adevice%2Frscb23-1)](https://computingforgeeks.com/wp-content/uploads/2021/06/manage-jenkins.png?ezimgfmt=ng%3Awebp%2Fngcb23%2Frs%3Adevice%2Frscb23-1)

Manage Plugins

[![manage plugins](https://computingforgeeks.com/wp-content/uploads/2021/06/manage-plugins-1024x475.png?ezimgfmt=rs:696x323/rscb23/ng:webp/ngcb23)](https://computingforgeeks.com/wp-content/uploads/2021/06/manage-plugins-1024x475.png?ezimgfmt=rs:696x323/rscb23/ng:webp/ngcb23)

Available Tab

<iframe id="google_ads_iframe_/1254144,21789414456/computingforgeeks_com-medrectangle-3_0" title="3rd party ad content" name="google_ads_iframe_/1254144,21789414456/computingforgeeks_com-medrectangle-3_0" scrolling="no" marginwidth="0" marginheight="0" style="border: 0px none; vertical-align: bottom;" sandbox="allow-forms allow-popups allow-popups-to-escape-sandbox allow-same-origin allow-scripts allow-top-navigation-by-user-activation" srcdoc="" data-google-container-id="8" data-load-complete="true" width="580" height="400" frameborder="0"></iframe>



[![manage plugins search for sonarqube](https://computingforgeeks.com/wp-content/uploads/2021/06/manage-plugins-search-for-sonarqube-1024x349.png?ezimgfmt=rs:696x237/rscb23/ng:webp/ngcb23)](https://computingforgeeks.com/wp-content/uploads/2021/06/manage-plugins-search-for-sonarqube-1024x349.png?ezimgfmt=rs:696x237/rscb23/ng:webp/ngcb23)

Select “***SonarQube Scanner***” once it shows up in the list of plugins. Click on “***Install without restart\***” tab then wait for it to complete installing. Mine will not appear on  the screenshots shared because I have it already installed as shown  below.

[![show sonarqube scanner is installed](https://computingforgeeks.com/wp-content/uploads/2021/06/show-sonarqube-scanner-is-installed-1024x432.png?ezimgfmt=rs:696x294/rscb23/ng:webp/ngcb23)](https://computingforgeeks.com/wp-content/uploads/2021/06/show-sonarqube-scanner-is-installed-1024x432.png?ezimgfmt=rs:696x294/rscb23/ng:webp/ngcb23)

## Step 2: Generate Token in SonarQube Server

In this step we are going to generate a token that we will use in Jenkins  server to connect to it. We already covered the installation of  SonarQube Server so head over to your installed instance, log in as *Administrator* and do the following:

<iframe style="border: 0px none; vertical-align: bottom;" src="https://1b93cc32d81da81bfe2ab05caa6653a6.safeframe.googlesyndication.com/safeframe/1-0-38/html/container.html" id="google_ads_iframe_/1254144,21789414456/computingforgeeks_com-medrectangle-4_0" title="3rd party ad content" name="" scrolling="no" marginwidth="0" marginheight="0" data-is-safeframe="true" sandbox="allow-forms allow-popups allow-popups-to-escape-sandbox allow-same-origin allow-scripts allow-top-navigation-by-user-activation" data-google-container-id="a" data-load-complete="true" width="300" height="250" frameborder="0"></iframe>



Click on your ***Admin account Icon\*** which will bring up a drop down menu. Click on “***My Account***“.

[![sonarqube my account](https://computingforgeeks.com/wp-content/uploads/2021/06/sonarqube-my-account-1024x303.png?ezimgfmt=rs:696x206/rscb23/ng:webp/ngcb23)](data:image/svg+xml,)

That will open a new page. On the new page, click on the “***Security\***” tab.

[![sonarqube my account security tab](https://computingforgeeks.com/wp-content/uploads/2021/06/sonarqube-my-account-security-tab-1024x536.png?ezimgfmt=rs:696x364/rscb23/ng:webp/ngcb23)](data:image/svg+xml,)

A new “***Tokens***” area will appear. Under “***Generate Tokens\***“, put a name you like then hit “***Generate\***“. 

<iframe id="google_ads_iframe_/1254144,21789414456/computingforgeeks_com-box-4_0" title="3rd party ad content" name="google_ads_iframe_/1254144,21789414456/computingforgeeks_com-box-4_0" scrolling="no" marginwidth="0" marginheight="0" style="border: 0px none; vertical-align: bottom;" srcdoc="" data-google-container-id="b" data-load-complete="true" width="1" height="1" frameborder="0"></iframe>



[![sonarqube my security generate token](https://computingforgeeks.com/wp-content/uploads/2021/06/sonarqube-my-security-generate-token-1024x523.png?ezimgfmt=rs:696x355/rscb23/ng:webp/ngcb23)](data:image/svg+xml,)

A new token will be generated. Copy it since we are going to use it in Jenkins next.

## Step 3: Configure SonarQube in Jenkins

While still logged into Jenkins as an administrator go to “***Manage Jenkins\***” > “***Configure System\***“.

[![manage jenkins 1](https://computingforgeeks.com/wp-content/uploads/2021/06/manage-jenkins-1-1024x378.png?ezimgfmt=rs:696x257/rscb23/ng:webp/ngcb23)](data:image/svg+xml,)

[![configure system](https://computingforgeeks.com/wp-content/uploads/2021/06/configure-system-1024x498.png?ezimgfmt=rs:696x338/rscb23/ng:webp/ngcb23)](data:image/svg+xml,)


Scroll down to the ***SonarQube\*** configuration section, click ***Add SonarQube\***, and add the values you will be prompted to provide.

[![configure system add SonarQube](https://computingforgeeks.com/wp-content/uploads/2021/06/configure-system-add-SonarQube-1024x477.png?ezimgfmt=rs:696x324/rscb23/ng:webp/ngcb23)](data:image/svg+xml,)

These include a name you prefer, SonarQube server URL, that is where your SonarQube server is running at then the “***Server authentication token\***“. For the Server authentication token, click on “***Add\***” then “***Jenkins\***” as shown below.

<iframe style="border: 0px none; vertical-align: bottom;" src="https://1b93cc32d81da81bfe2ab05caa6653a6.safeframe.googlesyndication.com/safeframe/1-0-38/html/container.html" id="google_ads_iframe_/1254144,21789414456/computingforgeeks_com-banner-1_0" title="3rd party ad content" name="" scrolling="no" marginwidth="0" marginheight="0" data-is-safeframe="true" sandbox="allow-forms allow-popups allow-popups-to-escape-sandbox allow-same-origin allow-scripts allow-top-navigation-by-user-activation" data-google-container-id="c" data-load-complete="true" width="336" height="280" frameborder="0"></iframe>



[![jenkins add sonarqube token 1](https://computingforgeeks.com/wp-content/uploads/2021/06/jenkins-add-sonarqube-token-1-1024x362.png?ezimgfmt=rs:696x246/rscb23/ng:webp/ngcb23)](data:image/svg+xml,)

That will open up the Jenkins Credentials Provider. Leave the domain as it is, and choose “***Secret Text\***” for Kind as shown below. Once that is done, enter the token we created in Step 2 for “***Secret\***“, give it a ***Name/ID\*** that will match with what the secret is all about, you can add a description if you like then hit “***Add\***“.

[![jenkins add sonarqube token 2](https://computingforgeeks.com/wp-content/uploads/2021/06/jenkins-add-sonarqube-token-2.png?ezimgfmt=rs:696x361/rscb23/ng:webp/ngcb23)](data:image/svg+xml,)

After that is done, scroll through “***Server authentication token\***” and choose the token we have just added. Once that is done, simply hit “***Apply\***” then “***Save\***“.

<iframe id="google_ads_iframe_/1254144,21789414456/computingforgeeks_com-large-leaderboard-2_0" title="3rd party ad content" name="google_ads_iframe_/1254144,21789414456/computingforgeeks_com-large-leaderboard-2_0" scrolling="no" marginwidth="0" marginheight="0" style="border: 0px none; vertical-align: bottom;" sandbox="allow-forms allow-popups allow-popups-to-escape-sandbox allow-same-origin allow-scripts allow-top-navigation-by-user-activation" srcdoc="" data-google-container-id="d" data-load-complete="true" width="250" height="250" frameborder="0"></iframe>



[![jenkins add sonarqube token 3 choose it](https://computingforgeeks.com/wp-content/uploads/2021/06/jenkins-add-sonarqube-token-3-choose-it-1024x524.png?ezimgfmt=rs:696x356/rscb23/ng:webp/ngcb23)](data:image/svg+xml,)

## Step 4: Configure SonarQube Scanner

In order for our code to be scanned by SonarQube, we need to configure SonarQube scanner. To do this, head over to “***Manage Jenkins\***” then click on “***Global configuration Tool\***“.

[![manage jenkins 2](https://computingforgeeks.com/wp-content/uploads/2021/06/manage-jenkins-2-1024x378.png?ezimgfmt=rs:696x257/rscb23/ng:webp/ngcb23)](data:image/svg+xml,)

[![adding sonarscanner global config tool](https://computingforgeeks.com/wp-content/uploads/2021/06/adding-sonarscanner-global-config-tool-1024x479.png?ezimgfmt=rs:696x326/rscb23/ng:webp/ngcb23)](data:image/svg+xml,)

Scroll down and look for “***SonarQube Scanner\***“. Click on “***Add SonarQube Scanner\***” tab so that we can add it here. Since we already have a working instance of SonarQube, uncheck “***Install Automatically\***” and then beside “***SONAR_RUNNER_HOME\***” environment variable, enter the path where SonarQube is installed in your server. For me it is “*/opt/sonarqube/bin*“. After that, hit “***Apply\***” then “***Save\***“.

[![adding sonarscanner 1](https://computingforgeeks.com/wp-content/uploads/2021/06/adding-sonarscanner-1-1024x520.png?ezimgfmt=rs:696x353/rscb23/ng:webp/ngcb23)](data:image/svg+xml,)

## Step 5: Create a Project in SonarQube

Head over to your SonarQube Server, log in as Administrator and create a project by following the following steps. Click on “***Administration\***” > “***Projects\***“. Click on the “***Projects\***” drop down list and click on “***Management\***“. On your far right, close to the top, you will see a tab called “***Create Project\***“.

[![sonarqube scanner create poject in sonarqube](https://computingforgeeks.com/wp-content/uploads/2021/06/sonarqube-scanner-create-poject-in-sonarqube-1024x526.png?ezimgfmt=rs:696x358/rscb23/ng:webp/ngcb23)](data:image/svg+xml,)

Click on it and it will bring a form where you will enter the details of your project, that is its *name* and *key*. You can key in any value here as you prefer then click on “***Create\***“.

[![sonarqube scanner create poject in sonarqube hit create](https://computingforgeeks.com/wp-content/uploads/2021/06/sonarqube-scanner-create-poject-in-sonarqube-hit-create-1024x488.png?ezimgfmt=rs:696x332/rscb23/ng:webp/ngcb23)](data:image/svg+xml,)

Your Project will be successfully crated as shown below.

[![sonarqube scanner create poject in sonarqube created](https://computingforgeeks.com/wp-content/uploads/2021/06/sonarqube-scanner-create-poject-in-sonarqube-created-1024x295.png?ezimgfmt=rs:696x201/rscb23/ng:webp/ngcb23)](data:image/svg+xml,)

## Step 6: Do a test with a Java Project

Guess what, our integration is finally done. Just one more thing remaining,  and that is testing it out with sample code. I will use code written in  SpringBoot and package it using Maven after scanning it using SonarQube. We shall add a step in a Jenkinsfile that will enable us scan the  application in SonarQube before it proceeds to other steps. The stage we will use for this in our pipeline is shared below.

```
stage ('Scan and Build Jar File') {
            steps {
               withSonarQubeEnv(installationName: 'Production SonarQubeScanner', credentialsId: 'SonarQubeToken') {
                sh 'mvn clean package sonar:sonar'
                }
            }
        }
```

1. *installationName* : SonarQube Server in Jenkins 
2. *credentialsId* : *Name/ID\* of the token we added when we were integrating SonarQube and Jenkins in *Step 3*.

After adding that stage, save your Jenkinsfile or push it to Git or other  source control integrated with your Jenkins Server and build your  project if you have not configured a webhook. Once the build is done,  the SonarQube stage should appear in your stages as follows:

[![jenkins pipeline with sonarqube stage](https://computingforgeeks.com/wp-content/uploads/2021/06/jenkins-pipeline-with-sonarqube-stage-1024x233.png?ezimgfmt=rs:696x158/rscb23/ng:webp/ngcb23)](data:image/svg+xml,)

Below the pipeline, you should see SonarQube Scan results similar to the one shared below

[![jenkins sonarqube scan results](https://computingforgeeks.com/wp-content/uploads/2021/06/jenkins-sonarqube-scan-results-1024x276.png?ezimgfmt=rs:696x188/rscb23/ng:webp/ngcb23)](data:image/svg+xml,)

Simialrly, you can view more details of the scan we have just completed by logging in to SonarQube an checking out the name of our project. In case there  are code smells, vulnerabilities or outdated packages, you will be able  to view it all therein.

[![sonarqube scan results](https://computingforgeeks.com/wp-content/uploads/2021/06/sonarqube-scan-results-1024x408.png?ezimgfmt=rs:696x277/rscb23/ng:webp/ngcb23)](data:image/svg+xml,)

#### Video Courses For Learning Jenkins:

- [Jenkins, From Zero To Hero: Become a DevOps Jenkins Master](https://click.linksynergy.com/deeplink?id=2sb89XJC/EQ&mid=39197&murl=https%3A%2F%2Fwww.udemy.com%2Fcourse%2Fjenkins-from-zero-to-hero%2F)
- [Learn DevOps: CI/CD with Jenkins using Pipelines and Docker](https://click.linksynergy.com/deeplink?id=2sb89XJC/EQ&mid=39197&murl=https%3A%2F%2Fwww.udemy.com%2Fcourse%2Flearn-devops-ci-cd-with-jenkins-using-pipelines-and-docker%2F)
- [Jenkins 2 Bootcamp: Fully Automate Builds to Deployment](https://click.linksynergy.com/deeplink?id=2sb89XJC/EQ&mid=39197&murl=https%3A%2F%2Fwww.udemy.com%2Fcourse%2Fjenkins-continuous-integration-bootcamp%2F)
- [DevOps Project: CI/CD with Jenkins Ansible Docker Kubernetes](https://click.linksynergy.com/deeplink?id=2sb89XJC/EQ&mid=39197&murl=https%3A%2F%2Fwww.udemy.com%2Fcourse%2Fvalaxy-devops%2F)

## Concluding Remarks

We have managed to not only install Jenkins and SonarQube but we have been able to integrate them so that our code goes through a good and smooth  transition in a DevSecOps manner till the end. Jenkins is a brilliant  tool and it has many more plugins you can play around with to make your  DevOps experience even better.

Thank You for reading through as we hope that the guide was as helpful as we intended it to be.

[How To Setup your Heroku PaaS using CapRover](https://computingforgeeks.com/setup-your-own-heroku-paas-using-caprover/)

[How To Validate CloudFormation Templates with cfn-lint and cfn-nag](https://computingforgeeks.com/validate-cloudformation-templates-with-cfn-lint-nag/)

[How To Create AWS EFS Filesystem With CloudFormation](https://computingforgeeks.com/create-aws-efs-filesystem-with-cloudformation/)