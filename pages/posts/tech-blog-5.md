---
title: How to use AWS Lambda to trigger “any” script as an API call
date: 2024/7/10
description: Guides and Steps to Run Scripts on AWS Lambda for Beginners
tag: Tech Blog
author: You
---

![MainImage](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*SUH_2pNRgWBmryk3970E5w.png)

First and foremost, **What is AWS Lambda?**

> AWS Lambda is a serverless, event-driven compute service that lets you run code for virtually any type of application or backend service without provisioning or managing servers — AWS Website

**In simple terms,**

Service that runs your code without you thinking about servers. You pay only for the compute time you consume and the number of requests.

**Why AWS Lambda?**

One million monthly requests are free; you pay very little for millions more requests. The possibilities with AWS Lambda are limitless. And AWS is constantly adding new features to AWS Lambda. I encourage you to try using AWS Lambda.

![](https://cdn-images-1.medium.com/max/1200/1*BepKVG5FODH_ybESySvN8A.png)

Today, we’re going to explore **AWS Lambda functions** step by step. We’ll start by making a simple function that talks to an **API using HTTPS**. Then, we’ll learn how to create a **unique URL for our function** and use it to make our script do its thing. Excited? Let’s dive in!

---

#### Let’s Learn by Doing:

**Step 1 — Create your AWS Lambda function:**

Head to your [AWS Console](https://console.aws.amazon.com/console/home), search for AWS Lambda, and click “Create Function”. You will see a below screen where you can provide basic details such as Function Name, Runtime language, etc.

**I am choosing Node.js for the example.**

![](https://cdn-images-1.medium.com/max/1200/1*PwNecCVTW3-3wY_6cvRcHw.png)

**Step 2 — Understand the basic options and run your “Hello from Lambda!”**

The lambda interface provides various options to **run, test and monitor** your code.

> **Code —** *A Simple interface to write down your script, equipped with language syntax highlighting.*

> **Test —** *A way to invoke your function for testing, you can provide a sample JSON event to the function and can also save your test events here.*

> **Monitor —** *Lambda logs all requests your function handles and automatically stores logs generated by your code through Amazon CloudWatch Logs. Lambda also sends runtime metrics for your functions to Amazon CloudWatch. These all can be monitored using this.*

Now that you have learned the basics and have successfully created your Lambda function, it’s time to run the first “Hello from Lambda” code.

![](https://cdn-images-1.medium.com/max/1200/1*X9LSzTnMa9Osc3b2yfGxBw.png)

See that pre-loaded code in the function saying “Hello from Lambda!”? To bring it to life, just hit the “Test” button, name it, and watch the magic happen. Your function will spit out a response, and voila! You’ve just made your Lambda flex its digital muscles.

**How’s that for a quick win?**

![](https://cdn-images-1.medium.com/max/1200/1*yzMjSOzTiHRmbbf5D0lp6A.png)

Time to take the show on the road! Invoke your script by calling the Lambda function URL.

**Are you excited to see this in action?**

---

#### Invoke a simple script by calling the Lambda function URL:

![](https://cdn-images-1.medium.com/max/1200/1*KxAC3QqqUiyzCRt278fcxA.png)

Let’s write a simple script that uses the **Axios** library to make an HTTPS GET call to a restcountries API. If successful, it returns the API response; otherwise, it gives an error message.

**Below is the script that I am using:**

```python
import axios from "axios";

async function apiCall(method, url) {
  var config = {
    method,
    maxBodyLength: Infinity,
    url,
    data: {}
  };
    
  return await axios(config)
  .then(function (response) {
    return response.data;
  })
  .catch(function (error) {
    console.log(error);
    return error;
  });  
}

export const handler = async(event) => {
  const url = 'https://restcountries.com/v3.1/name/india?fullText=true';
  return apiCall('GET', url);
};
```

Now that you have a script ready, we need to **create a function URL** for our lambda function. You can now create a lambda function URL with **Authorization as NONE**.

![](https://cdn-images-1.medium.com/max/1200/1*ECJ6iIDYHp0tnzvltXgcRA.png)

As we have our URL ready now, let’s invoke our script. I have used Postman to make an HTTPS GET call to the function URL.

![](https://cdn-images-1.medium.com/max/1200/1*hEfpfrv6C0GjzZLL9zrvfg.png)

And Voila!! That’s it. You have successfully created your first script on the Lambda function and invoked it by making a simple HTTPS GET call.

This simple script is just the beginning. Now, armed with the power of serverless computing, you’re ready to tackle even grander feats. Keep coding, keep experimenting, and let the Lambda adventures continue!

---

*Thanks for reading. If you like to read more such articles, I invite you to follow me.*

*Till then, Sayonara!!*