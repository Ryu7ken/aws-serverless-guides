<h2>Creating a Event Driven Architecture by uploading a .mp3 or .mp4 file to S3 Bucket which triggers Lambda to convert the audio into text using Transcribe</h2>

<h3>Below is the step by step guide to setup this architecture:</h3>

***Step-1:*** Go to Lambda console and create a function.

***Step-2:*** Give your function a name, select Python as the runtime and click on create function.

![Creating Function](/assets/images/CreatingFunction.png)

***Step-3:*** Put the below code in the Lambda function and click on deploy

```python
import boto3
import uuid

client = boto3.client("transcribe")

def lambda_handler(event, context):
    
    myid = uuid.uuid4().int
    bucketname = event["Records"][0]["s3"]["bucket"]["name"]
    filename = event["Records"][0]["s3"]["object"]["key"]
    
    client.start_transcription_job(
        TranscriptionJobName="transcribejob-" + str(myid),
        LanguageCode="en-IN",
        MediaFormat="mp3",
        Media={
            "MediaFileUri": "s3://"+bucketname+"/"+filename,
        },
        OutputBucketName="Provide_your_output_bucket_name",
    )
```
> [!IMPORTANT]
> Change the OutputBucketName in the Lambda code according to your S3 bucket name that you've to create for the Transcribe output files.

![Deploy Code](/assets/images/DeployCode.png)

***Step-4:*** Go to the Configuration of your Lambda function, then go to Permissions and click on the Role name and you will be redirected to IAM Roles console.

![Configuration](/assets/images/Configuration.png)

***Step-5:*** In the IAM Roles console, go to Permissions policies and click on Add permissions, then click on Attach policies.

![IAM Roles](/assets/images/IAMRoles.png)

***Step-6:*** Search for "AmazonTranscribeFullAccess" and "AmazonS3FullAccess" and tick them. Then click on Add premissions.

![IAM Roles](/assets/images/IAMRoles2.png)

***Step-7:*** Go to S3 Console and create 2 buckets, one for inputs which will triger the Lambda function and one to get output of Transcribe. Give unique names to your buckets and click on "Create bucket".

![Create Bucket](/assets/images/Createbucket.png)

![Create Bucket](/assets/images/Createbucket2.png)

***Step-8:*** Go to your S3 bucket that in which you want to input your files, then go to "Properties" and then scroll down to "Event notifications", then click on "Create event notification".

![Create Event](/assets/images/Createevent.png)

***Step-9:*** Give your event a name, then in "Event types" click on "All object create events". Then at "Destination" select "Lambda function" and select the Lambda function that you had created and click "Save changes".

![Create Event](/assets/images/Createevent2.png)

***Step-10:*** Now upload your mp3 file to the S3 bucket that you created for input. And you will get the output of Transcribe in the S3 bucket that you had mentioned in the Lambda code in "OutputBucketName".

![Transcribe Output](/assets/images/Output.png)

<h3>Now you can convert any audio form mp3 or mp4 into a text file with this architecture.</h3>