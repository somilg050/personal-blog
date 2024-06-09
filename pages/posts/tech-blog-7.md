---
title: Tried Mistral Large on AWS Bedrock
date: 2024/7/10
description: Mistral's Latest Model Large is Now Available on AWS
tag: Tech Blog
author: You
---

![MainImage](https://miro.medium.com/v2/resize:fit:2000/format:webp/1*hE7Q648mR5eKdha7SeFoWw.png)

On 2nd April, AWS announced the availability of **Mistral Large on Amazon Bedrock**. I thought of giving it a spin. You can read more about the announcement [here](https://aws.amazon.com/about-aws/whats-new/2024/04/mistral-large-foundation-model-amazon-bedrock/)[.](https://aws.amazon.com/about-aws/whats-new/2024/04/mistral-large-foundation-model-amazon-bedrock/)

#### What is Mistral and Mistral Large?

Mistral is a French AI startup famous for releasing high-performance LLM models as open source.

Mistral Large is Mistral's latest model with benchmark performance that exceeds Claude 2.1. Widely known for its top-tier reasoning capabilities, specific instruction following, and multilingual translation abilities.

---

**Let's get the access 💪**

Head to the AWS Bedrock console to use the Mistral Large model. To use Bedrock, you must request access to Bedrock's FMs. Once you apply for the request, you will see the Access status as "Access Granted" in a few minutes, as shown in the image below.

![](https://cdn-images-1.medium.com/max/1800/1*nSeOyfEbK2hTTOziHNme7A.png)

Now that we have access, let's dig deeper into the code.

In this tutorial, let's create a basic text summarizer using the Mistral Large model. That will take in some long-form content as input and give us three short summaries as output.

First and foremost, we will set up our AWS SDK for Python using Boto3 and AWS CLI. If you have not installed them before —

```bash
(base) ➜  ~ pip3 install boto3
(base) ➜  ~ pip3 install awscli

(base) ➜  ~ aws configure
```

Let's set up the Bedrock client using the boto3.

```python
import boto3

bedrock = boto3.client(service_name="bedrock-runtime",
                       region_name='us-east-1')
```

Below is the prompt we will use to create a text summarizer, which also tells us the confidence of the output.

```python
prompt = """
<s>[INST]You are a summarization system that can provide summaries with 
associated confidence scores. In clear and concise language, provide 
three short summaries of the following essay, along with their confidence
scores. You will respond with a pretty response with Summary and Confidence. 
Do not provide explanations.[/INST]

# Essay: 
{Essay}
"""
```

Let's quickly implement the UI using Streamlit to interact with our model. You can customize the Streamlit UI however you want; I have created a basic UI to demonstrate the use case.

You can play around with the LLM parameters to see the changes in the output.

```python
modelId = "mistral.mistral-large-2402-v1:0"

def streamlit_ui():
    st.set_page_config("Mistral<>Bedrock")
    st.header("Text Summarization using Mistral's Large and AWS Bedrock")

    user_question = st.text_area("Provide me with a text to summarize.")

    if st.button("Summarize") or user_question:
        if not user_question:
            st.error("Please provide a text to summarize.")
            return
        with st.spinner("Processing..."):
            filled_prompt = prompt.format(Essay=user_question)
            body = json.dumps({
                "prompt": filled_prompt,
                "max_tokens": 512,
                "top_p": 0.8,
                "temperature": 0.5,
            })
            response = bedrock.invoke_model(
                body=body,
                modelId=modelId,
                accept="application/json",
                contentType="application/json"
            )
            response_json = json.loads(response["body"].read())
            text = response_json['outputs'][0]['text']
            st.write(text)
            st.success('Done!')
```

This is how our Streamlit application will look.

![](https://cdn-images-1.medium.com/max/1800/1*9LLMRrQMvkBAb9UJMMU2gg.png)

In the example below, I summarized one of my [medium stories](https://medium.com/illumination/the-people-of-vietnam-535304dd45fb).


Check the result, which consists of three summaries, as mentioned in the prompt, along with the confidence score from the model. The result seems pretty good to me.

![](https://cdn-images-1.medium.com/max/1800/1*AZtl6JU7q3IrQvvXKdNpoA.png)

The complete code for the application is available here on my github: [somilg050](https://github.com/somilg050/rag-aws-bedrock/blob/master/using_mistral_large.py).

---

*Thanks for reading the tutorial. I hope you learn something new today. If you want to read more stories like this, I invite you to follow me.*

*Till then, Sayonara! I wish you the best in your learning journey.*

*I am a software engineer who enjoys travelling and writing —* [***About Me***](https://medium.com/about-me-stories/about-me-somil-gupta-cc8ba13bdd8c)***.***