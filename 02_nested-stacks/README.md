# Running the nested stacks sample
There are two ways to run the nested stack sample;

## Using AWS console
1. Create a S3 bucket (Public Read) and upload the nested stacks (any file that starts with prefix greater than 01).
1. Update the S3 object HTTP URL in the parent stack against the TemplateURL property.
1. Create a cloudformation stack by uploading the root stack.
   > You may have to update your S3 bucket's policy to get it working. Check how the below CLI approach behaves before creating a bucket policy.

## Using AWS CLI
1. Change directory into the 02_nested-stacks.
1. Run the below command to create a S3 bucket
   ```bash
   aws s3api create-bucket --bucket {your-bucket-name} --acl public-read
   ```
1. Run the below command to package your nested stacks.
   ```bash
   aws cloudformation package --s3-bucket {bucket-name-from-step2} --template-file 01_parent.yaml --output-template-file 01_parent-packaged.yaml
   ```
1. Complete the proces by running the below command
   ```bash
   aws cloudformation deploy --template-file 01_parent-packaged.yaml --stack-name tempa-nested --parameter-overrides ProjectName=nestedproj Environment=Dev --capabilities CAPABILITY_IAM
   ```