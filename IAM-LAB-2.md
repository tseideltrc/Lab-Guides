# <span style="color:orange">LAB 10 - Advanced Roles and Groups Management using IAM</span>

## <span style="color:green">Introduction</span>

An IAM role is an IAM identity that you can create in your account that has specific permissions. An IAM role is similar to an IAM user, in that it is an AWS identity with permission policies that determine what the identity can and cannot do in AWS. However, instead of being uniquely associated with one person, a role is intended to be assumable by anyone who needs it. Also, a role does not have standard long-term credentials such as a password or access keys associated with it. Instead, when you assume a role, it provides you with temporary security credentials for your role session.

---

## <span style="color:green">Instructions</span>

### Task 1: Creating a Customer Managed Policy with Policy Generator

1. On the **Identity & Access Management** dashboard, click **Policies** on the left.

2. Click **Create Policy**.

3. Select **S3** for the **Service** and **ListAllMyBuckets** for the **Action**

4. Skip the tagging step, and click on **Review Policy** and in Policy Name field insert *policy-lab*.

5. Choose **Create Policy** to save your new policy.

---

### Task 2: Attaching a Policy to Users

1. On the **Identity & Access Management** dashboard, click **Policies** on the left.

2. Type *AmazonS3ReadOnlyAccess* into the search box then select the only available policy

3. Click **Policy Actions** and then choose **Attach**.

4. Select the **John** User to attach the policy.

5. Click **Attach Policy**.

---

### Task 3: Creating an IAM Role

1. From the **Identity & Access Management** dashboard, click **Roles**.

2. Click **Create role**.

3. On the **Create Role** page, select **AWS service** under **Select type of trusted entity** and **EC2** under C**hoose a use case**, then click **Next: Permissions**.

4. On the **Attach permissions policies** page, filter for the name *S3Full* and select the policy *AmazonS3FullAccess*.

5. Click **Next: Tags** and then **Next: Review**.

6. Set the **Role name** to *lab-role*.

7. Click **Create Role**.

8. Click on the role you created to inspect its details.

---

### Task 4: Launching EC2 Instances with IAM Profile

In AWS, you can designate an IAM role to attach to an EC2 instance when launching the instance, or any time after. Attaching an IAM role to an instance allows you to manage permissions for instances centrally with IAM. In this Lab Step, you'll practice launching an EC2 instance with an IAM role attached.

1. In the AWS Management Console search bar, enter *EC2*, and click the **EC2** result under **Services**.

2. From the **EC2** dashboard, click **Launch Instance**.

3. Select the **Amazon Linux 2** AMI at the top of the list.

4. On the **Choose an Instance Type** page, click on **Next: Configure Instance Details**.

5. On the **Configure Instance** tab, select **lab-role** for the **IAM Role** and then click **Review and Launch**.

6. On the **Review Instance Launch** page, click **Launch**.

7. In the **Select an existing key pair or create a new key pair** dialog box, select **Choose an existing key pair** if you already created a key pair in this region. Otherwise, select **Create a new key pair**. 

8. Select the acknowledgment checkbox and then click **Launch Instances**.

9. Click **View Instances** to return to the **EC2** dashboard.

---

### Task 5: Connecting to a Remote Shell Using an SSH Connection

In order to manage a remote Linux server, you must employ an SSH Client. Secure Shell (SSH) is a cryptographic network protocol for securing data communication. It establishes a secure channel over an insecure network. Common applications include remote command-line login and remote command execution. In this Lab Step, you will practice connecting to an EC2 instance. Follow the steps for either Linux / macOS or Windows to connect to your instance, depending on which OS you are currently using

1. Locate your .pem key file on your computer. 

2. In the AWS Management Console search bar, enter *EC2*, and click the **EC2** result under **Services**.

3. Click **Instances** and highlight the EC2 instance you want to connect to and click **Connect**.

4. Copy the example ssh command on the **SSH Client** tab to your clipboard. The ssh command can be found after you click **Connect**.

5. Open your Terminal application on your local computer (i.e. GitBash, Terminal, iTerm).

6. If this is the first time you using this keypair: In your Terminal application, navigate to the directory containing your .pem key and enter the following command to place the required level of security on the key pair in order to use it for ssh: 

  * `chmod 400  /home/youruser/keypair.pem`

> *Note*: To navigate to the directory containing your .pem key, use the **cd** command. For example, if your .pem key was downloaded to your Downloads directory, you might run a command similar to this:

  * `cd ~/user/Downloads`

> where user is your username on your local computer. 

7.  Paste the command you copied from the EC2 dashboard in order to ssh into the server.

---

### Task 6: Testing IAM from an EC2 Linux Instance

Instead of creating and distributing your AWS credentials,  you can use IAM roles to delegate permission for making API requests. In this Lab Step, you will practice using IAM instance profiles to manage permissions in an EC2 instance as well as using an IAM user's credentials to do the same.

1. Use the following command to verify your EC2 instance has the correct instance profile:

  * `curl http://169.254.169.254/latest/meta-data/iam/info`

  > The command should return a JSON object with an **InstanceProfileArn** value ending in **lab-role**. If it does you have launched your EC2 instance with the correct instance profile.

2. Run the following commands to test that you can create and S3 buckets, making sure to replace "randomnumber" with a random number:

  * `aws s3 mb s3://aw-academy-testbucket-randomnumber`

  * `aws s3 ls`

> So far you used the IAM instance profile attached to the instance to perform these actions. Now you will call the AWS API by specifying the Access Key and Secret Key of the John user in the command line. You will generate the required keys, by performing the following steps:

3. Navigate to the IAM dashboard and click **Users**.

4. Click **John**.

  * You will use John to understand what happens whether an unauthorized operation is made (you will try to create an S3 bucket. John is not authorized to do so)

5. Select the **Security Credentials** tab to generate security credentials for John.

6. Click **Create access key**.

7. Click **Show** to reveal the **Secret access key**.

8. Copy the access key and secret access key for future use.

  * You could also use the access key and secret access key in the CSV file you downloaded when you created the John user.

9. In your SSH session enter these commands:

  * `export AWS_ACCESS_KEY_ID=access-key-that-you-saved`
  * `export AWS_SECRET_ACCESS_KEY=secret-key-that-you-saved`

  * This will configure the EC2 instance for the user, John. The EC2 instance will use the provided security credentials and override the instance profile set to the instance.

10. Run the following commands: 

  * `aws ec2 describe-instances --region us-west-2`
  * `aws s3 mb s3://aw-academy-labs-test-bucket-randomnumber`

  * Notice that while you can list EC2 instances now due to the permissions set for John, you no longer have the ability to create S3 buckets because the instance profile has been overridden. To check which identity is making API calls use the `aws sts get-caller-identity` command.
