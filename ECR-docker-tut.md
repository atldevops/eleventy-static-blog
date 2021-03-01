### How to push a docker image to AWS ECR
This tutorial will outline the steps required to push a docker image to Amazon's Elastic Container Registry from the command line. Once the initial steps are done pushing your images to ECR will be much faster.
Prerequisites:
AWS account
Access key id and secret access key
Docker image

Steps:
Configure AWS CLI
Create AWS repository/Set lifecyle policy
Retrieve Password and authenticate to registry
Set up IAM permissions to allow user access to registry
Add tags to image
Push image to Amazon Elastic Container Registry

Let's get started
First we need to configure our AWS CLI. You will need to enter your Access Key ID and Secret Access Key for your user profile. As well as your default region. Leave the default output format blank as it will default to JSON.

aws configure
2. Next, create your repository in AWS.
aws ecr create-repository --repository-name NAMEOFYOURREPO
Output:
3. Now we will create a lifecycle policy to clean up older images once the count gets to 800. Make sure to add your own registry id and repository name.
aws ecr put -lifecycle-policy --registry-id YOURREGISTRYID --repository-name YOURREPONAME --lifecycle-policy-text '{"rules" [{"rulePriority":10,"description":"Expire old images","selection":{"tagStatus":"any","countType":"imageCountMoreThan","countNumber":800},"action":{"type":"expire"}}]}'
Output:
4. Next we will need to authenticate the docker cli to our default registry. This allows docker to push and pull images with Amazon ECR. Make sure to edit the two region sections and the aws_account_id with the corresponding information from your account.
aws ecr get-login-password \
    --region <region> \
| docker login \
    --username AWS \
    --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
5. Almost there, let's set up our IAM permissions to allow user access to the registry. Take note of account username you will need to edit.
Note: For the sake of the tutorial I am going to grant full access to the ECR repositories. This would not be a best practice in production as you would want to create groups, attach the the correct policy to the group,and then add the necessary users to the group.
aws iam attach-user-policy --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryFullAccess --user-name john_doe
6. Tag Image your image. The registry format is aws_account_id.dkr.ecr.region.amazonaws.com. The repository name should match the repository that you created for your image. If you leave out the image tag, Amazon will add the tag "latest" to your image.
The example below will tag the image with id 5c3aaak6cek73 as aws account 1234567789.dkr.ecr.us-east-1.amazonaws.com/atldevops.
docker tag YOURIMAGEID AWS_ACCOUNT_ID.dkr.ecr.REGION.amazonaws.com/TAG
7. Last we can push our image, push the image using the docker push command. The format is
docker push AWS_ACCOUNT_ID.dkr.ecr.REGION.amazonaws.com/TAG
All done here. Check your Elastic Container Registry in the console to verify that your images have been pushed to your registry.