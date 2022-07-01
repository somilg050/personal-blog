---
title: CI & CD pipeline using AWS Code Pipeline and Elastic Beanstalk
date: 2022/7/01
description: Build a fully CI/CD application.
tag: Tech Blog
author: You
---

### CI & CD pipeline using AWS Code Pipeline and Elastic Beanstalk

![Image](https://miro.medium.com/max/1400/1*9uGuGEt-GLX4XNVfjmtlbQ.png)

**What is AWS Code Pipeline?**
As per the AWS official website, AWS CodePipeline is a fully managed continuous delivery service that helps you automate your release pipelines for fast and reliable application and infrastructure updates. CodePipeline automates the build, test, and deploy phases of your release process every time there is a code change, based on the release model you define.

Now, **what is AWS Elastic Beanstalk?**
Again as per the website, AWS Elastic Beanstalk is an easy-to-use service for deploying and scaling web applications and services.
In simple words upload your code and Elastic Beanstalk automatically handles the deployment, from capacity provisioning, load balancing, and auto-scaling to application health monitoring.

Let's get it started...

![Image](https://s3.ap-south-1.amazonaws.com/somilgupta.me-docs/Screenshot+2022-07-01+at+7.32.11+AM.png)

So for creating an application head over to elastic beanstalk and click "Create Application".
Provide the basic details such as the app name, platform (e.g. Node.js, Java, go etc.), select the platform version and then click on "Create Environment".

This will start the provisioning of resources for your application (creating ec2 instance, security group, log groups etc.), meanwhile, grab something to eat and relax as this will take a few minutes.

After it's done, you will see a page like this. Which shows your sample application has been successfully created.

![Image](https://s3.ap-south-1.amazonaws.com/somilgupta.me-docs/Screenshot+2022-07-01+at+7.41.33+AM.png)

Now our first task is done. We have created a sample EBS application. The second task on our bucket list is to create a code pipeline for CI/CD.

Now head over to the AWS CodePipeline service.

**Step 1:** Select "Create Pipeline", and give a beautiful name to your pipeline.  
**Step 2:** **"Most Important One"** Add source stage
Select Github version 2 here and do the basic authentication using your github account.
Provide your source repo and the branch that you want to deploy.

**Step 3:** **"Again a important one"** Add deploy stage
As we are using EBS as the environment provided for our application, make sure you select EBS and the exact environment name.

As you can see how we have connected all of the three services together in step2 and step3.

Now hit "Create pipeline."
This will create a pipeline for you, something like this. Which shows the Github in source stage and our AWS EBS in deploy stage.

![Image](https://s3.ap-south-1.amazonaws.com/somilgupta.me-docs/Screenshot+2022-07-01+at+7.53.08+AM.png)

Just a sample image for you to understand :)

Now the final and most awaited step...
Just kidding everything is done and you are good to go now.

So whenever you push any changes in your GitHub repo, the pipeline will start the auto deployment on the EBS stack. And using the environment url in your EBS application you can test your APIs, see UI changes etc.

That's it!!

Thanks for reading folks :)