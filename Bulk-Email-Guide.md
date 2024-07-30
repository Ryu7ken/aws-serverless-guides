<h2>Creating a Lambda function to retrieve Email IDs from object in S3 bucket and bulk Email using SES</h2>

<h3>Below is the step by step guide to setup this architecture:</h3>

***Step-1:*** Lambda console, create a function by giving a name and selecting Python as the runtime. Then click on "Create function".

![Createfunc](/assets/images/Createfunc.png)

***Step-2:*** Copy the below code and paste in the Lambda function and click on "Deploy".

```python
import boto3
import os

def lambda_handler(event, context):
    # Get the bucket and object key from the event
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Initialize S3 client
    s3 = boto3.client('s3')
    
    # Retrieve the object from S3
    response = s3.get_object(Bucket=bucket, Key=key)
    emails_content = response['Body'].read().decode('utf-8')
    
    # Extract email addresses
    email_addresses = emails_content.splitlines()
    
    # Initialize SES client
    ses = boto3.client('ses')
    
    # Send email to each address
    for email in email_addresses:
        response = ses.send_email(
            Source='enter_sender_email',
            Destination={
                'ToAddresses': [
                    email,
                ]
            },
            Message={
                'Subject': {
                    'Data': 'Test Email from Lambda',
                },
                'Body': {
                    'Text': {
                        'Data': 'This is a test email sent from AWS Lambda using Amazon SES.',
                    }
                }
            }
        )
    
    return {
        'statusCode': 200,
        'body': 'Emails sent successfully!'
    }
```

> [!IMPORTANT]
> Make sure to change  Source='enter_sender_email' and enter the email address you want to use to send email.

![Deployses](/assets/images/Deployses.png)

***Step-3:*** Go to the "Configuration" tab of the Lambda function and then go to "Permissions" tab, then click on the "Role name" and you will be redirect to IAM Roles console.

![Config](/assets/images/Config.png)

***Step-4:*** At the "Permissions policies" section, click on the "Add permissions" and then click on "Attach policies".

![Config2](/assets/images/Config2.png)

***Step-5:*** Serach for "AmazonS3ReadOnlyAccess" and "AmazonSESFullAccess" and select them. Then click on "Add permisssions".

![Config3](/assets/images/Config3.png)

![Config4](/assets/images/Config4.png)

![Config5](/assets/images/Config5.png)

***Step-6:*** In the "General configuration" section, click on "Edit" and change the "Timeout" to 1 Minute.

![Timeout](/assets/images/Timeout.png)

![Timeout2](/assets/images/Timeout2.png)

***Step-7:*** Go to SES Console, then go to "Identities" section and click on "Create identity".

![SES](/assets/images/SES.png)

***Step-8:*** Then select identity type as "Email Address" and enter email address. Then click on Create identity.

![SES2](/assets/images/SES2.png)

> [!IMPORTANT]
> Create Identity for the Sender's Email and all the Reciever's Email. Otherwise the SES Service would stop the Lambda function from sending emails.

***Step-9:*** Go to S3 Console and click on "Create bucket".

![S3Create](/assets/images/S3Create.png)

***Step-10:*** Give a unique name to your S3 bucket and click on "Create bucket".

![S3Create2](/assets/images/S3Create2.png)

![S3Create3](/assets/images/S3Create3.png)

***Step-11:*** Now click on the S3 bucket that you created and click on the "Properties" tab.

![EventConfig](/assets/images/EventConfig.png)

***Step-12:*** Scroll down and click on "Create event Notification".

![EventConfig2](/assets/images/EventConfig2.png)

***Step-13:*** Give the event a name and in the "Event types" section select "All object create events".

![EventConfig3](/assets/images/EventConfig3.png)

***Step-14:*** In the "Destination" section select "Lambda function" and select the Lambda function you created a while ago. Then click on "Save changes".

![EventConfig4](/assets/images/EventConfig4.png)

***Step-15:*** Now create a .txt file in your local system and put Email IDs in it.

![Emails](/assets/images/Emails.png)

***Step-16:*** Upload the .txt in the S3 bucket and emails will be sent the Email IDs provided in the .txt files if they were verified in the SES Service as mentioned in **Step-8**.

![S3Upload](/assets/images/S3Upload.png)

![S3Upload](/assets/images/S3Upload2.png)

![S3Upload](/assets/images/S3Upload3.png)

<h3>Now you can send bulk emails using this architecture.</h3>