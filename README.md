# Connecting-AWS-EC2-application-with-S3-bucket
In this blog, we will learn how to connect S3 bucket from an EC2 instance. In a single-tier architecture (Traditional web hosting), an application’s files and static assets are hosted in the same server/instance. This would make users feel slow sometimes during high load and the traffic is at its peak.

To improve the speed and performance while keeping the cost factors in mind, we can host the application’s images in the S3 bucket and connect the application to fetch images from the S3 bucket directly.

In this way, image requests are handled by S3 and only the application is served by the EC2 instance. This would accommodate some more users on the website by the EC2 instance and improve the business performance.

  - Architectural Diagram
![blog](https://github.com/user-attachments/assets/a78187cc-9a74-4f04-8a1d-9d1ac23d2103)

Let us see how to make this work.
 - Deploying the application in the EC2 instance.
 - Creating an AWS IAM user with an S3FullAccess policy.
 - Configuring the IAM user in EC2 AWS CLI.
 - Create an AWS S3 bucket.
 - Syncing Images from EC2 to S3 via AWS CLI.
 - Adding Apache redirection of image requests to S3.
 - Making S3 bucket public via S3 bucket policy.
 - Understanding, images are served by S3.

Considering that we have an EC2 instance running with httpd installed.

## Apache/httpd installation steps
~~~
sudo yum install httpd -y //For Amazon linux, CentOS, RHEL based

sudo apt update; apt install apache2 -y //For ubuntu

sudo systemctl restart httpd.service

sudo systemctl enable httpd.service
~~~

## Step 1: Deploying the application in the EC2 instance 🚀

We are going to deploy a website template with images on the EC2 instance.

Download the zip file of the template using the wget command

Unzip the zipped file

Copy the unzipped directory to /var/www/html/ directory which Apache’s default document root

Change the permission of all the files and sub-directories to apache:apache

![image](https://github.com/user-attachments/assets/09e38ae1-85df-417e-90b4-e9e38875918f)

The public DNS of the instance works fine with images in the same instance.

![image](https://github.com/user-attachments/assets/10331ca8-20a6-4ef1-bb64-24c43e5ce97c)

## Step 2: Creating an AWS IAM user with S3FullAccess policy👤

We are leveraging the use of AWS IAM roles to allow privileges for EC2 to access the S3 service.

Navigate to the IAM console >> click on Users at the left side menu >> Add users

![user](https://github.com/user-attachments/assets/b024e6aa-e5de-449c-a480-fb35b8d7dfca)

![user](https://github.com/user-attachments/assets/49f3dbfc-d54a-4b7a-ae3e-6a5b6b7e14d6)

- Username: s3user

### Disable Providing user access to the AWS Management Console

![user](https://github.com/user-attachments/assets/bfb78641-56a2-4b59-8fa3-2eaa94f56098)

To provide the user S3FullAccess permission, click on Attach policies directly so that the IAM user can list objects, get objects, etc.

![user](https://github.com/user-attachments/assets/ccf21a71-efe7-4376-88f9-45561a6cf781)


![user](https://github.com/user-attachments/assets/b190cb03-fc78-4229-9518-85b9a69518fa)

In the IAM users page, you can see the list of IAM users that you have created.

![user](https://github.com/user-attachments/assets/535a0699-f711-45dd-a944-f93af84bb5ee)

Click on the user s3user >> Security credentials tab >> Under Access keys, Click on Create access key

![user](https://github.com/user-attachments/assets/0853d084-5ad9-429d-9cba-bed3920d8fc4)

Select **other** in **Access key best practices & alternatives**

![image](https://github.com/user-attachments/assets/64f0388b-d9c0-4573-a2ea-266f95b085e8)

**Retrieve access keys**: Copy the access key and secret access key to a safe location.

**Note: A secret access key cannot be retrieved once a user is created. Hence, it is important to copy and save the access key and secret access key.**

![user](https://github.com/user-attachments/assets/31d45d89-be60-4431-bcd4-5021dd0b45c9)

## Step 3: Configuring the IAM user in EC2 AWS CLI

SSH into the EC2 instance where you have deployed the application.

Using the **aws configure** command, we are going to provide the access key and secret access key of the s3user IAM user to use the programmatic access of s3user.

![image](https://github.com/user-attachments/assets/df7fa6af-dd5b-4717-917d-8cde13cef06c)

## Step 4: Create an AWS S3 bucket

Navigate to the AWS S3 console >> Click on **Create bucket**

**General configuration**

**Bucket name:** app.jeethu.shop
**AWS Region:** ap-south-1

**Object ownership:** Ensure ACLs disabled

![user](https://github.com/user-attachments/assets/11c19c87-86e0-4023-9b4c-5a11c2471b5b)

### Disable Block all public access

![user](https://github.com/user-attachments/assets/67849df7-86ec-4e23-aece-6f279fe0d45d)

**Bucket versioning:** Disabled

![user](https://github.com/user-attachments/assets/1cf11cc3-27d4-4653-aa64-32179e6bd807)

## Step 5: Syncing Images from EC2 to S3 via AWS CLI

In this section, we are going to sync/copy the /var/www/html/images directory from the EC2 instance to the destination S3 bucket named app.jeethu.shop

Here, we are using the sync command so that if the S3 bucket has any existing files, that won't be overwritten by the sync command.

 **sync vs cp command of AWS S3**

 `aws s3 sync` command will scan the destination location and only overwrite the file from the source location if the file is newly created or updated. It is **idempotent.**

 The S3 bucket does not have any files at present.

 ![image](https://github.com/user-attachments/assets/6bf63f77-2f29-4ce8-83ff-0cc0c200c71e)

 Using the command **aws s3 sync /var/www/html/images/ s3://app.jeethu.shop** to copy the files from the local directory of the EC2 instance to the destination bucket.

 ![user](https://github.com/user-attachments/assets/6966f7ee-c188-4b93-bdb1-9199522852c0)

 The S3 bucket got the image files now which can be seen over the AWS S3 console or awscli command `aws s3 ls s3://app.jeethu.shop`

 ![image](https://github.com/user-attachments/assets/0955bf9f-2539-4229-8423-74f023664680)

 ![image](https://github.com/user-attachments/assets/951edc12-e576-4076-ad15-4cdd7032ba55)

### Step 6: Adding Apache redirection of image request to S3

By default, apache serves the images from the /var/www/html/images directory. We are going to edit the httpd.conf file to inform apache to redirect the image requests to the s3 bucket’s public URL.

Edit the httpd.conf file `vim /etc/httpd/conf/httpd.conf`

Add the redirection rule and save the httpd.conf file

Check the Apache configuration syntax test using `httpd -t`

Restart apache `systemctl restart httpd`

![image](https://github.com/user-attachments/assets/aaa3f6ca-a45a-4e97-8fe9-ef84d6f6e17e)

![user](https://github.com/user-attachments/assets/4b46edd0-455f-41cc-aea6-6131d3e31fbf)

![image](https://github.com/user-attachments/assets/0201fc62-e3b5-4572-a88d-67c677a66d9e)

As soon, as we redirected the image requests to the S3 bucket, the website did not show the images.

![image](https://github.com/user-attachments/assets/ad9289a0-6c98-4cd2-9a30-523a69f61af8)

The reason behind the images not loading is that even though the bucket is made public, we need to enable public internet users to access permission on the specific bucket.

It is necessary to make the bucket objects to be accessible through the bucket policy.

## Step 7: Making S3 bucket public via S3 bucket policy

In this section, we will see how to add a bucket policy.

S3 console >> Click on the bucket app.jeethu.shop >> Permissions tab

![user](https://github.com/user-attachments/assets/9591b131-629c-4989-982b-9188cdf352a9)

**Bucket policy >> Edit >> Add the bucket policy >> Save**

![image](https://github.com/user-attachments/assets/b2589e4d-e25b-42f7-a574-07a60690da94)

![image](https://github.com/user-attachments/assets/e12ec09f-f5cd-41c5-bdc8-867420507c3a)

The Bucket policy says to allow **(“effect”: “allow”)** serving the objects to all users **(“Principal”:** *) from the bucket (“Resource”: “arn:aws:s3:::app.jeethu.shop/*”)

### Step 8: Understanding images are served by S3 

Now the bucket is made public for internet users who access it through GET requests. The website has also started showing images.

![image](https://github.com/user-attachments/assets/e003a3a5-faf1-4c60-af72-0403dee7a0e2)

I have confirmed that the images are served by S3 by right clicking the image >> Click on open image in new tab >> S3 public URL is serving the image request.

![user](https://github.com/user-attachments/assets/5f88a5a9-cfdf-4770-a49c-a4444865000a)

With this setup,**web application requests are handled by the EC2 instances, and image requests are handled by S3 buckets which reduces significant load on the server and improves application speed.**


Hope the blog is useful! 🌟 See you soon!! 👋
