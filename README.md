# GitHub Actions auto deploy of docker containers on ECR & App Runner

## Prerequisites

 - Having an AWS account
 - Having a GitHub account
 - Having **docker** and **aws** installed on your machine

## Index

1. Create a GitHub repo
2. Create ECR repository
3. Upload docker image to ECR
4.  Create AppRunner service
5. Create IAM role for ECR
6. Create GitHub Action

## 1. Create a GitHub repo
-   Go to your GitHub profile, and then click on "**Repositories**". Click on the green button "**New**", give the repo a name (***ECR-AppRunner-deploy***) and then choose Public or Private based on your needs; then click on the "**Create repository**" green button. Once you have created it, copy the link, go to your terminal and enter the command: `git clone [link-to-your-repo.git]`.
- Create your server files and your **Dockerfile**.

## 2. Create ECR repository
 - In you AWS  account, go to the **ECR control panel** in the section **Repositories**: https://console.aws.amazon.com/ecr/repositories
 - Choose the appropriate **region** (***eu-west-1***)
 - Click on **"Create repository"**, give it a name (***app-be***) and click on **"Create repository"**.

## 3. Upload docker image to ECR
 - Once created the **ECR repository**, you should be in the repository page. Click on **"View push commands"**. Run the four commands on your machine, at the location of your Dockerfile. The commands should be:
	 - `aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin <accountID>.dkr.ecr.<region>.amazonaws.com`, where you need to replace **\<region>** and **\<accountID>**

## 4. Create AppRunner service

 - In you AWS  account, go to the **AppRunner control panel**: https://console.aws.amazon.com/apprunner
 - Click on **"Create service"**
 - Paste the URI of the container image or **Browse** it
 - Under **Deployment settings**, select Automatic, the click on **"Next"**. If you don't have the appropriate role, click on **"Create new service role"**, then give it a name or leave as default. If existing, click on **"Use existing service role"** and select it. Then click on **"Next"**
 - Give the service a name (***app-be***) and eventually **add environment variables**
 - Set the desired **port** (the same specificed in the **Dockerfile**)
 - Click on **"Next"**, review the service and then click on **"Create & deploy"**

## 5. Create IAM user for ECR
-   In your AWS account, go to the **IAM control panel**, in the section **Users**: https://console.aws.amazon.com/iam/home#/users
-  Create a new user:
    - click on "**Add user**" button, then give the user a name (***Github-Action-AWL-CLI-Allow-ECR***) and, under the "Select AWS access type" section, check the "**Programmatic access**" box; then, click on "**Next: Permissions**"
    - In "Set permissions" section, click on "**Attach existing policies directly**", and then find the following policy:
        - **AmazonEC2ContainerRegistryFullAccess**
    - Then click on "**Next: Tags**" and then on "**Next: Review**". Be sure that everything's ok and then click on "**Create user**".
    - **IMPORTANT**: now you'll see the ***Access key ID*** and the ***Secret access key*** for the User you just created. We need to set them as GitHub Secrets. Don't close this window until the next steps aren't completed, or you'll lose these credentials forever, making it necessary to repeat the entire User creation procedure.
        - In your GitHub repo, go to the "**Settings**" panel, then go to "**Secrets**";
        -  Click on "**New repository secret**": give the secret a name (`AWS_ACCESS_KEY_ID`) and paste the value you see in the AWS User page. Then click on "**Add Secret**";
        - Do the same for the Secret access key: give it a name (`AWS_SECRET_ACCESS_KEY`), paste the value and then "**Add Secret**".

## 6. Create GitHub Action
 -   In your project folder, **create a folder** and name it `.github`. Mind the dot before github. Inside that folder, **create another folder** called `workflows`. Inside that folder, **create a file** called `deploy.yml` containing the following (eventually replace **region** and **ECR_REPOSITORY**):
```
name: ECR deploy CI/CD pipeline
  on:
  push:
    branches: [ master ]
		
jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
      uses: actions/checkout@v2
		
      - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: eu-west-1
		
      - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1
	
      - name: Build, tag, and push the image to Amazon ECR
      id: build-image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        ECR_REPOSITORY: app-be
        IMAGE_TAG: latest
      run: |
        # Build a docker container and push it to ECR
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
        echo "Pushing image to ECR..."
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        echo "::set-output name=image::$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG"
```
 - Push your commits to run the GitHub Action.
