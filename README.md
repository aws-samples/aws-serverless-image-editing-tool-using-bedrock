# Deploy a serverless web application to edit images using Amazon Bedrock

[Generative AI](https://aws.amazon.com/generative-ai/) adoption among various industries is revolutionizing all different types of applications, including image editing. Image editing is used in various sectors, such as graphic designing, marketing, and social media. Users rely on specialized tools for editing images. Building a custom solution for this task can be complex. However, by using various AWS services, you can quickly deploy a [serverless](https://aws.amazon.com/serverless/) solution to edit images. This approach can give your teams access to image editing foundation models (FMs) using [Amazon Bedrock](https://aws.amazon.com/bedrock/).

Amazon Bedrock is a fully managed service that makes FMs from leading AI startups and Amazon available through an API, so you can choose from a wide range of FMs to find the model that’s best suited for your use case. Amazon Bedrock is serverless, so you can get started quickly, privately customize FMs with your own data, and integrate and deploy them into your applications using AWS tools without having to manage any infrastructure.

[Amazon Titan Image Generator G1](https://docs.aws.amazon.com/bedrock/latest/userguide/titan-image-models.html) is an AI FM available with Amazon Bedrock that allows you to generate an image from text, or upload and edit your own image. Some of the key features we focus on include [inpainting](https://docs.aws.amazon.com/bedrock/latest/userguide/titan-image-models.html#titanimage-features:~:text=an%20image%20mask.-,Inpainting,-%E2%80%93%20Uses%20an%20image) and [outpainting](https://docs.aws.amazon.com/bedrock/latest/userguide/titan-image-models.html#titanimage-features:~:text=with%20background%20pixels.-,Outpainting,-%E2%80%93%20Uses%20an%20image). As of the writing of this blog post, Amazon Titan Image Generator G1 comes in two versions; for this blog post we will use version 2 of the model.

This blog post will introduce a solution that simplifies the deployment of a web application for image editing using AWS serverless services. We will use [AWS Amplify](https://aws.amazon.com/amplify/), [Amazon Cognito](https://aws.amazon.com/cognito/), [Amazon API Gateway](https://aws.amazon.com/api-gateway/), [AWS Lambda](https://aws.amazon.com/lambda/), and [Amazon Bedrock](https://aws.amazon.com/bedrock/) with the [Amazon Titan Image Generator G1](https://docs.aws.amazon.com/bedrock/latest/userguide/titan-image-models.html) model to build an application that can be used to edit images using prompts. We will cover the inner workings of the solution to help you understand the function of each service and how they are connected to give you a complete solution.

## Solution overview

Now let’s go through the solution and how the AWS services are connected and used for this application. The following diagram provides an overview and highlight the key components. The architecture utilizes Cognito for user authentication and Amplify as the hosting platform for our front-end application. A combination of API Gateway and Lambda function is used for our back-end services and Amazon Bedrock integrates with the FM model, enabling users to edit the image using prompts.

\[Screenshot\]

## Prerequisites

You must have the following in place to complete the solution in this post.

1. An [AWS account](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fportal.aws.amazon.com%2Fbilling%2Fsignup%2Fresume&client_id=signup)
2. Enable foundation model [access](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html) in Amazon Bedrock for Amazon Titan Image Generator G1 v2 in the same AWS Region where you will deploy this solution.
3. Download CloudFormation script from [aws-samples Github](https://github.com/aws-samples/aws-serverless-image-editing-tool-using-bedrock) page.

### Resources created after running the CloudFormation template

When you run the [AWS CloudFormation](https://aws.amazon.com/cloudformation) template, the following resources will be deployed.

- Amazon Cognito
  - [User pool](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools.html): CognitoUserPoolforImageEditApp
  - [App client](https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-settings-client-apps.html): ImageEditApp
- Lambda
  - [Function](https://docs.aws.amazon.com/lambda/latest/dg/getting-started.html): &lt;Stack name&gt;-ImageEditBackend-&lt;auto-generated&gt;
- [AWS Identity Access Management (IAM)](https://aws.amazon.com/iam/)
  - [IAM role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html): &lt;Stack name&gt;-ImageEditBackendRole-&lt;auto-generated&gt;
  - [IAM inline policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#inline-policies): AmazonBedrockAccess–Allows Lambda to invoke Amazon Bedrock foundation model amazon.titan-image-generator-v2:0
- Amazon API Gateway
  - [Rest API](https://docs.aws.amazon.com/apigateway/latest/developerguide/rest-api-develop.html): ImageEditingAppBackendAPI
  - [Methods](https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-method-settings.html):
    - OPTIONS – Added header mapping for [CORS](https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-cors.html)
    - POST – Lambda integration
  - [Authorization](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-enable-cognito-user-pool.html): Through Amazon Cognito using CognitoAuthorizer

After successfully deploying the CloudFormation template, copy the following from the output section to be used during the deployment of Amplify. An example is shown in the following screenshot.

- userPoolId
- userPoolClientId
- invokeUrl

\[Screenshot\]

### Resources deployed manually

You will have to manually deploy the Amplify application using the frontend code found on GitHub.

  1. Download the frontend code from the [GitHub repo](https://github.com/aws-samples/aws-serverless-image-editing-tool-using-bedrock).
  2. Unzip the downloaded file and navigate to the folder.
  3. In the js folder, find the config.js file and replace the values of XYZ for userPoolId, userPoolClientId, and invokeUrl with the values provided in the [Output](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/outputs-section-structure.html) tab after deploying the CloudFormation template. Set the region value based on the Region where you're deploying the solution.

Example config.js file:

:::code{showCopyAction=true showLineNumbers=true language=python}
window._config = {
    cognito: {
        userPoolId: 'XYZ', // e.g. us-west-2_uXboG5pAb
        userPoolClientId: 'XYZ', // e.g. 25ddkmj4v6hfsfvruhpfi7n4hv
        region: 'XYZ' // e.g. us-west-2
    },
    api: {
        invokeUrl: 'XYZ' // e.g. https://rc7nyt4tql.execute-api.us-west-2.amazonaws.com/prod,
    }
};
:::

\[Screenshot\]

   4. Select all the files and compress them as shown in the following image. Make sure you zip the contents and not the top-level folder. For example, if your build output generates a folder named AWS-Amplify-Code, navigate into that folder and select all the contents, and then zip the contents as shown in the following image.

\[Screenshot\]

 5. Use the new .zip file to manually [deploy](https://docs.aws.amazon.com/amplify/latest/userguide/manual-deploys.html) the application in Amplify. Once it is deployed you will receive a domain which you can use in later steps to access the application.

\[Screenshot\]

   6. [Create](https://docs.aws.amazon.com/cognito/latest/developerguide/how-to-create-user-accounts.html#creating-a-new-user-using-the-console) a test user in the Amazon Cognito user pool.  
        **Note:** An email address is required for this user because you will need to mark email address as verified.

\[Screenshot\]

  7.  Return to AWS Amplify page and use the domain it auto generated to access the application.

### Amazon Cognito for user authentication

[Amazon Cognito](https://docs.aws.amazon.com/cognito/latest/developerguide/what-is-amazon-cognito.html) is an identity platform that you can use to authenticate and authorize users. We use Cognito in our solution to verify the user before they can use the image editing application.

Upon accessing the Image Editing Tool URL, you will be prompted to sign in with a previously created test user. For first-time sign-ins, users will be asked to update their password. After this process, the user's credentials are validated against the records stored in the user pool. If the credentials match, Amazon Cognito will issue a JSON Web Token (JWT). In the **API payload to be sent** section of the page, you will notice that the **Authorization** field has been updated with the newly issued JWT.

### Lambda for backend code and Amazon Bedrock for generative AI function

The backend code is hosted on Lambda and launched by user requests routed through API Gateway. The Lambda function will process the request payload and forward it to Amazon Bedrock. The reply from Amazon Bedrock will follow the same route as the initial request.

### Amazon API Gateway for API management

API Gateway streamlines API management, allowing developers to deploy, maintain, monitor, secure, and scale their APIs effortlessly. In our use case, API Gateway serves as the orchestrator for the application logic and provides throttling to manage the load to the backend. Without API Gateway, we would need to use the JavaScript SDK in the frontend to interact directly with the Amazon Bedrock API, bringing more work to the frontend.

### Amplify for front-end code

Amplify is a service that offers a development platform for building secure, scalable mobile and web applications. It allows developers to focus on their code rather than to worry about the underlying infrastructure. Amplify also integrates with many Git providers. For this solution, we manually upload our frontend code using the method outlined in the **Resources deployed manually** section.

### Image editing tool walkthrough

Navigate to the URL provided after creating the application in Amplify and sign in.

\[Screenshot\]

As you follow the steps for this tool, you will notice the **API Payload to be Sent** section on the right side updating dynamically, reflecting the details mentioned in the corresponding steps that follow.

**Step 1: Create a mask on your image**

1. Choose a file (JPEG, JPG, or PNG).
2. After the image is loaded, the frontend converts the file into base64 and base_image value is updated.
3. As you select a portion of the image you want to edit, a mask will be created, and mask value is updated with a new base64 value. You can also use the stroke size option to adjust the area you are selecting.
4. You now have the original image and the mask image encoded in base64.

**Note:** Amazon Titan Image Generator G1 model requires the inputs to be in base64 encoding.

\[Screenshot\]

**Step 2: Write a prompt and set your options**

1. Write a prompt that describes what you want to do with the image. In the preceding figure, we enter Make the driveway clear and empty. and this is reflected in the prompt on the right.
2. Image editing options – inpainting and outpainting: mode is updated depending on your selection.
   - Use inpainting to remove masked elements and replace them with background pixels.
   - Use outpainting to extend the pixels of the masked image to the image boundaries.
3. Choose **Send to API** button to send the payload to the API gateway. This action will invoke the Lambda function, which validates the received payload. If the payload is validated successfully, the Lambda function proceeds to invoke the Amazon Bedrock API for further processing.

The Amazon Bedrock API will generate two image outputs in base64 format, which will be transmitted back to the frontend application and rendered as visual images.

**Step 3: View and download the result**

The following figure shows the results of our test. You can download the results or provide an updated prompt to get a new output.

\[Screenshot\]

## Testing and troubleshooting

When you initiate the Send to API action, the system performs a validation check. If any required information is missing or incorrect, it will display an error notification. For instance, if you attempt to send an image to the API without providing a prompt, an error message will appear on the right side of the interface, alerting you to the missing input, as shown in the following figure.

\[Screenshot\]

## Clean up

If you decide to discontinue using the Image Editing Tool, you can follow these steps to remove the Image Editing Tool, its associated resources deployed using CloudFormation, and the AWS Amplify deployment:

1. Navigate to the AWS CloudFormation console.
2. Locate the stack you created during the deployment process (you assigned a name to it).
3. Select the stack and choose **Delete**.
4. Follow these [instructions](https://aws.amazon.com/getting-started/hands-on/build-web-app-s3-lambda-api-gateway-dynamodb/module-six/) to delete the Amplify application and its resources by selecting the name of the application you used during deployment.

## Conclusion

In this blog post, we explored a sample solution that you can use to deploy an image editing application by using AWS Serverless services and generative AI services. We used Amazon Bedrock and an Amazon Titan FM that allows you to edit images by using prompts. By adopting this solution, you gain the advantage of using AWS managed services, eliminating the need for maintaining any underlying infrastructure. Get started today by deploying this sample solution.

## Additional resources

To learn more about Amazon Bedrock, see the following resources:

- [Amazon Bedrock Workshop](https://github.com/aws-samples/amazon-bedrock-workshop)
- [Amazon Bedrock User Guide](https://docs.aws.amazon.com/bedrock/latest/userguide/what-is-bedrock.html)
- [Amazon Bedrock with Amazon Titan Image Generator G1](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-titan-image.html)
- [Amazon Bedrock InvokeModel API](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_InvokeModel.html)
- [Using generative AI on AWS for diverse content types Workshop](https://catalog.us-east-1.prod.workshops.aws/genai-on-aws/)

To learn more about the Titan Image Generator G1 model, see the following resources:

- [Amazon Titan Image Generator G1 model](https://docs.aws.amazon.com/bedrock/latest/userguide/titan-image-models.html)
- [Amazon Titan Image Generator Demo](https://www.youtube.com/watch?v=v2akUur4xho)

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

