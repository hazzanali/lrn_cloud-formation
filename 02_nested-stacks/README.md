# Running the nested stacks sample
There are two ways to run the nested stack sample;

## Using AWS console
1. Create a S3 bucket and the nested stack (*02-1_instance.yaml* and *02-1_network.yaml*).
1. Update the S3 object HTTP URL in the parent stack against the TemplateURL property.

## Using AWS CLI
1. Change directory into the 02_nested-stacks.
1. Run the below command to create a S3 bucket
   ```bash
   aws s3 create-bucket --bucket nested-stack-20210302 --acl public-read
   ```
1. Run the below command to package your nested stacks.
   ```bash
   aws cloudformation package --s3-bucket {bucket-name-from-step2} --template-file 01_parent.yaml --output-template-file 01_parent-packaged.yaml
   ```
1. Complete the proces by running the below command
   ```bash
   aws cloudformation deploy --template-file 01_parent-packaged.yaml --stack-name tempa-nested --parameter-overrides ProjectName=nestedproj Environment=Dev --capabilities CAPABILITY_IAM
   ```