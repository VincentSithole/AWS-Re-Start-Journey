Entered:

AWS Access Key ID: [provided]

AWS Secret Access Key: [provided]

Default region name: us-west-2

Default output format: json

Step 2: Create S3 Bucket
Created a new S3 bucket named cafe-luxury-test using the AWS CLI mb (make bucket) command.

bash
aws s3 mb s3://cafe-luxury-test --region 'us-west-2'
Output:

text
make_bucket: cafe-luxury-test
https://Screenshot%25202026-05-21%2520143009.png

Step 3: Upload Images to S3 Bucket
Synced sample images from the local initial-images/ folder to the S3 bucket under the images/ prefix.

bash
aws s3 sync ~/initial-images/ s3://cafe-luxury-test/images
Output:

text
upload: initial-images/Strawberry-Tarts.jpg to s3://cafe-luxury-test/images/Strawberry-Tarts.jpg
upload: initial-images/Cup-of-Hot-Chocolate.jpg to s3://cafe-luxury-test/images/Cup-of-Hot-Chocolate.jpg
upload: initial-images/Donuts.jpg to s3://cafe-luxury-test/images/Donuts.jpg
https://Screenshot%25202026-05-21%2520143009.png

Step 4: Review IAM Permissions
Reviewed the IAM group mediaco and user mediacouser.

Group: mediaco

Attached Policies:

IAMUserChangePassword – Allows users to change their own password

mediaCoPolicy – Custom inline policy for S3 access

mediaCoPolicy allows:

GetObject, PutObject, DeleteObject on cafe-*/images/*

Console bucket listing

User: mediacouser (member of mediaco group)

https://Screenshot%25202026-05-21%2520151810.png

Step 5: Test mediacouser Permissions
Signed in to AWS Management Console as mediacouser (password: Training1!).

Action	Result
View objects (opened Donuts.jpg)	✅ Allowed
Upload new object	✅ Allowed
Delete object (Cup-of-Hot-Chocolate.jpg)	✅ Allowed
Change bucket permissions	❌ Access Denied
Conclusion: IAM policy correctly restricts mediacouser to read/write/delete on images/ folder only.

Step 6: Create SNS Topic and Subscription
Created an SNS topic to receive email notifications when S3 bucket contents change.

Topic Name: s3NotificationTopic

Subscription:

Protocol: Email

Endpoint: phakamilesithole80@gmail.com

Confirmed subscription via email link.

https://Screenshot%25202026-05-21%2520144723.png

Step 7: Configure S3 Event Notifications
Created s3EventNotification.json to trigger SNS notifications on object creation and deletion within the images/ folder.

json
{
  "TopicConfigurations": [
    {
      "TopicArn": "arn:aws:sns:us-west-2:896038352564:s3NotificationTopic",
      "Events": ["s3:ObjectCreated:*", "s3:ObjectRemoved:*"],
      "Filter": {
        "Key": {
          "FilterRules": [
            {"Name": "prefix", "Value": "images/"}
          ]
        }
      }
    }
  ]
}
Applied the configuration to the S3 bucket:

bash
aws s3api put-bucket-notification-configuration --bucket cafe-luxury-test --notification-configuration file://s3EventNotification.json
Step 8: Test S3 Event Notifications
Reconfigured AWS CLI with mediacouser credentials and performed the following tests.

Test A: Upload Object (PUT)
bash
aws s3api put-object --bucket cafe-luxury-test --key images/Caramel-Delight.jpg --body ~/new-images/Caramel-Delight.jpg
Output:

json
{
  "ETag": "\"31ac30da619244b0ce786f106e4f3df7\"",
  "ServerSideEncryption": "AES256"
}
Result: ✅ Email received – ObjectCreated:Put for images/Caramel-Delight.jpg

Test B: Get Object (No Notification Expected)
bash
aws s3api get-object --bucket cafe-luxury-test --key images/Donuts.jpg Donuts.jpg
Result: ✅ No email received (correct – only create/delete events trigger)

Test C: Delete Object
bash
aws s3api delete-object --bucket cafe-luxury-test --key images/Strawberry-Tarts.jpg
Result: ✅ Email received – ObjectRemoved:Delete for images/Strawberry-Tarts.jpg

Test D: Unauthorized ACL Change
bash
aws s3api put-object-acl --bucket cafe-luxury-test --key images/Donuts.jpg --acl public-read
Output:

text
An error occurred (AccessDenied) when calling the PutObjectAcl operation
Result: ❌ Access Denied (expected – mediacouser cannot modify permissions)

https://Screenshot%25202026-05-21%2520151639.png

Step 9: Verify Final Bucket Contents
Opened the S3 console to confirm all uploaded files are present in the images/ folder.

https://Screenshot%25202026-05-21%2520151736.png

Results Summary
Objective	Status
Use s3api and s3 CLI commands to create and configure S3 bucket	✅
Verify write permissions to a user on an S3 bucket	✅
Configure event notification on an S3 bucket	✅
Task	Status
Create S3 bucket via CLI	✅
Upload images to S3	✅
mediacouser can view/upload/delete	✅
mediacouser cannot change bucket permissions	✅
SNS topic created and subscribed	✅
S3 event notification configured	✅
Upload triggers email notification	✅
Delete triggers email notification	✅
Get does not trigger email	✅
Unauthorized ACL change blocked	✅
Environment Details
Item	Value
AWS Account ID	8960-3835-2564
Region	us-west-2
S3 Bucket	cafe-luxury-test
SNS Topic ARN	arn:aws:sns:us-west-2:896038352564:s3NotificationTopic
IAM Group	mediaco
IAM User Tested	mediacouser
Screenshots
Filename	Description
Screenshot 2026-05-21 143009.png	Bucket creation and image sync
Screenshot 2026-05-21 144723.png	SNS subscription creation
Screenshot 2026-05-21 151639.png	CLI put-object, get-object, AccessDenied
Screenshot 2026-05-21 151736.png	Final bucket contents in S3 console
Screenshot 2026-05-21 151810.png	IAM group mediaco with policies
References
AWS CLI S3 Commands

S3 Event Notifications

IAM Policies

