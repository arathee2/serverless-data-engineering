# Serverless Data Engineering

This repository hosts a serverless NLP pipeline used to analyze sentiment. It consists of two lambda functions: *producer* and *consumer*. 

### Producer
- This lambda function will read from a DyanmoDB table and send values to SQS. A trigger will need to be set up for this, I made it invoke every minute.

### Consumer 
- This lambda function will look up the contents of the message in SQS and will look up a corresponding Wikipedia article. A trigger will also need to be set for this, I made it invoke every time a message was delivered to SQS. The corresponding Wikipedia is then passed to AWS Comprehend where sentiment analysis is performed. Results are then saved to a S3 bucket.

[Original Reference - Noah Gift](https://github.com/noahgift/awslambda)

### Serverless Pipeline Flow
![Alt text](https://camo.githubusercontent.com/bb29cd924f9eb66730bbf7b0ed069a6ae03d2f1a/68747470733a2f2f757365722d696d616765732e67697468756275736572636f6e74656e742e636f6d2f35383739322f35353335343438332d62616537616638302d353437612d313165392d393930392d6135363231323531303635622e706e67 "Serverless Data Engineering Flow")

### How to use:
1. Set up your DynamoDB table, S3 bucket, SQS queue, and IAM roles to start. Doing this at the start makes potential debugging much easier. 


2. In Cloud9, locate the aws-explorer tab (on the left). Go to lambda, right click and select "Create SAM lambda application." You will need to specify the workspace, python version (I used 3.7), and template (hello-world). 


3. Modify `app.py` to include the code from the `app.py` files in my producer folder. Edit the code to ensure that your DynamoDB table and SQS queue names match.


4. Deploy the SAM lambda application using the following code:
```
sam build --use-container
sam deploy --guided
```
5. In AWS Lambda you should now see your producer app. Click the trigger and then click permissions. If you have an IAM role set up, assign it, otherwise you may add policies so that the producer lambda function will be able to run. 

6. Delete the original trigger and set up one using 'EventBridge.' You will want to create a new rule and write the "schedule expression". I used: rate(1 minute). 

7. When enabled, messages should populate in your SQS queue. You can disable this trigger until consumer is complete so that SQS stops populating.

8. For the consumer, repeat steps 2-5 using the `app.py` file located in the consumer directory. 

9. Delete the original trigger and create a SQS trigger that will pull from your SQS queue. 

10. Check CloudWatch logs to check for errors and your S3 bucket to view the output of the sentiment analysis performed. 

11. Shutdown triggers when finished.

---

### Additional Considerations
If you are using a micro EC2 instance, you may not have enough space. There are two options around this. 
1. Resize your instance using the `resize.sh` file. 
2. Delete existing irrelevant docker images on C9.
```
docker image ls # list docker images
docker image rm IMAGE_ID IMAGE_ID IMAGE_ID #you can delete multiple images at once
```