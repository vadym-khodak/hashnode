## Deploying a Django project on AWS Lambda using Serverless (Part 2)

After my previous blog post [Deploy Django App on AWS Lambda using Serverless (Part 1)](https://blog.vadymkhodak.com/deploy-django-app-on-aws-lambda-using-serverless-part-1), people started asking me about AWS infrastructure for Django projects, so I decided to share my experience with this. 

It is not obvious (for people who don't have enough experience with AWS resources) how to create and configure all the necessary AWS resources to deploy a Django project on AWS Lambda using Serverless. 

Here are a list of ways of how to do this:
- manually via the [AWS console](https://console.aws.amazon.com/console/home)
- automatically using [Terraform](https://www.terraform.io/)
- automatically using [AWS SDK](https://aws.amazon.com/sdk-for-python/)

In this blog post, I will show you how to do it manually via [AWS console](https://console.aws.amazon.com/console/home).

## Update configuration for existing AWS resource and create new ones


**Step 1**: Create your own AWS account (if you don't have one).
Here is [a link to a manual for creating and activating an AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/).


**Step 2**: Go to the [AWS Management Console](https://console.aws.amazon.com/console/home)

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620288323328/TCd6I26My.png)


**Step 3**: Select your region. In my case, it is `US East (N. Virginia)us-east-1`

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620288299714/EkwdprPdu.png)


**Step 4**: Update `Security Group` with rules for AWS RDS service (PostgreSQL)

- *Type `EC2` in the search bar and click on the `EC2` service in the search results*
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620288343122/4e3Ofc0bz.png)

- *Choose the `Security Groups` option in the `Network & Security` section*


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620288629215/A0S055gKa.png)

- *Click on your security group id*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620291117330/thbXdrOe7.png)

- *Click on `Edit inbound rules`*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620291210595/1ZpUcgchc.png)

- *Click on `Add rule`. Then, select the `PostgreSQL` option in the `Type` column. Next, choose the `Anywhere` option in the `Source` column. Finally, click on the `Save rules` button*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620291384959/G3ebMKnBF.png)

- *Go to the `Outbound rules` tab and add the same rules that were described in the previous section*

**Step 5**: Create an `IAM role`

- *Type `IAM` in the search bar and click on the `IAM` service in the search results*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620636425458/fMGrWm7Zk.png)

- *Click on `Roles` in the `IAM` side bar or in the `IAM dashboard`*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620637568031/JKTygqDJC.png)

- *Click on the `Create Role` button*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620637765372/SUNLHVwFu.png)

- *Select `AWS service`, `Lambda`, and click on the `Next: Permissions` button*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620637743531/QqZrUg1hS.png)

- *Type `Lambda` in the search bar, select the `AWSLambda_FullAccess` policy, click on the `Next: Tags` button*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620637936495/IOrzQTI9V.png)

- *Click on the `Next: Review` button*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620638106114/FgVR_NPJk.png)

- *Type `Role name`, `Role description`, and click on the `Create role` button*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620638260257/bjTiOQvxS.png)

- *Click on the created role name*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620641663209/GntQz6Uj6.png)

- *Click on the `Copy Role ARN` button*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620641765824/wd4rQAKez.png)

Next, you should add the role ARN to the Serverless configuration directly or using environment variables (for example `.env` file)

```env
ROLE=arn:aws:iam::<your-aws-account-id>:role/exec_lambda
```

**Step 6**: Create `S3 buckets` for static assets and deployment

- *Type `S3` in the search bar and click on the `S3` service in the search results*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620295754837/J0C9fvFW-.png)

- *Click on the `Create bucket` button*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620295871356/4l5nTIeEx.png)

- *Type `Bucket name`, select your `AWS region`, unselect `Block all public access` and click on the `Create bucket` button*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620634116004/-0qbct0w7.png)

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620296394230/SjSIH3d8T.png)

Then, you should repeat all the steps mentioned above to create an S3 bucket for deployment.

Next, you should add your bucket names to your Django and Serverless configurations directly or using environment variables (for example `.env` file)

```env
AWS_STORAGE_BUCKET_NAME='django-react-static-assets'
DEPLOYMENT_BUCKET='django-react-deployments'
```

**Step 7**: Create a `CloudFront distribution`

- *Type `CloudFront` in the search bar and click on the `CloudFront` service in the search results*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620296521232/jO-AFfk-R.png)

- *Click on the `Create Distribution` button*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620296615377/2VeGX_DXW.png)

- *Click on the `Get started` button*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620296729397/-wU2n9y4c.png)

- *Select your `S3 bucket`*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620296828608/hBX8FuIPr.png)

- *Select: `Yes` for `Restrict Bucket Access`, `Create a New Identity` for `Origin Access Identity`, `Yes, Update Bucket Policy` for `Grant Read Permissions on Bucket`, `HTTP and HTTPS` for `Viewer Protocol Policy`*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620297821541/KbWjnM1pd.png)

- *Type "some comment" (optional) and click on the `Create Distribution` button*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620297924642/D5GAxVcSz.png)

- *Go to the distributions list and copy the `Domain Name`*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620298241028/-pYeM6nmu.png)

Then, you should add CloudFront distribution `Domain Name` to your Django and Serverless configurations directly or using environment variables (for example `.env` file)

```.env
AWS_S3_CDN_DOMAIN="<domain-id>.cloudfront.net"
```
**Step 8**: Create `RDS`

- *Type `RDS` in the search bar and click on the `RDS` service in the search results*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620298335720/H-ysDK1f9.png)

- *Click on the `Create database` button*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620552244393/XLgmatyuP.png)

- *Select `Standard create`, `PostgreSQL`, and `Version` (in my example, it is PostgreSQL 12.5-R1)*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620552230437/3e7j77XK_.png)

- *Select `Free Tier` as a `Template`, fill in `DB instance identifier`,  `Master username`, `Master password`, `Confirm password`*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620552456818/rnnBuaaVs.png)

- *Select `Burstable classes (includes t classes)` and `db.t2.micro` for `DB instance class`, `General Purpose (SSD)` as `Storage type`, and `20` as `Allocated storage`, unselect `Enable storage autoscaling`*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620552792865/Ez6OK7jVM.png)

- *Skip the `Availability & durability` section*

- *Configure `Connectivity`*:
    - Select your default VPC as `Virtual private cloud (VPC)`
    > After a database is created, you can't change the VPC selection.
    - Select `default` as `Subnet group`
    - Select `Yes` for `Public access`
    - Select `Choose existing` for `VPC security group`, and select `default` for `Existing VPC security groups` section
    - Select  `Availability Zone` (in my example `us-east-1a`)

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620553127175/rFZoEvtYO.png)

- *Select `Password authentication` or any other you want to use as `Database authentication options`, unselect all check boxes in `Additional configuration` and type `Initial database name` (in my example, it is `django_aws`)*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620553965107/ahg7Hq3v-p.png)


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620554209446/vfBmk2x0L.png)

- *Click on the `Create database` button*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620554259315/qwiicxMxk.png)

- *Go to the `RDS` dashboard and click on `Databases` in the `RDS` side bar or on `DB instances`*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620642406834/-W3N8eUfG.png)

- *Click on the created database identifier*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620642512235/SjJQQBwdS.png)

- *Copy the database endpoint, the subnets, and the security groups*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1620642725680/Wsr3oXQFRK.png)

Then, you should add this info to your Django and Serverless configurations directly or using environment variables (for example `.env` file)

```
DB_HOST='django-aws.<db-domain-id>.us-east-1.rds.amazonaws.com'
DB_USER='<your-master-db-user>'
DB_PASSWORD='<password-for-your-master-db-user>'
DB_NAME='<your-db-name>'
SECURITY_GROUPS=sg-<security-group-id>
SUBNETS=subnet-<subnet-id>,subnet-<subnet-id>,subnet-<subnet-id>,subnet-<subnet-id>,subnet-<subnet-id>,subnet-<subnet-id>
```

> **NOTE**: This is just an example of AWS configuration I use in my example. You may use your own configuration.


## Automate managing your AWS infrastructure 

I showed you how to configure all the necessary AWS resources for a Django project manually using the [AWS Management Console](https://console.aws.amazon.com/console/home). There are some ways to automate this process. I'll show how to manage AWS resources using [Terraform](https://www.terraform.io/) (infrastructure as code) in my next blog post. Follow me on Twitter [@vadim_khodak](https://twitter.com/vadim_khodak) or on [LinkedIn](https://www.linkedin.com/in/vadym-khodak-0b1a05149/) so you do not miss the next posts.

