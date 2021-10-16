# WEB-SOLUTION-WITH-WORDPRESS

Before we start we need to have an environment to work with. I will we using my AWS account to create an EC2 instance with an Ubuntu Server.


Use link:
[Creating Red Hat instance in AWS](https://github.com/hectorproko/RepeatableSteps_tutorials/blob/main/AWS_ReHat_Instnace.md)


We need to know which **availability zone** this instance is in when attaching **EBS**. Go to the list of instances to check. My example shows (instance which I named) **Project6** is in availability zone **us-east-1c**
<br /> 
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/WEB-SOLUTION-WITH-WORDPRESS/main/Images/project6.png)
 <br>

I will now create Elastic Block Volumes by going to **Elastic Block Store** > **Volumes** > **Create Volume**
<br /> 
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/WEB-SOLUTION-WITH-WORDPRESS/main/Images/createVolume.png)
 <br>

Once prompted we'll change the size to 10GB, make sure **availability zone** is the same as instance. In my case **us-east-1c**. In addition, I will add a **tag** to name the volume
<br /> 
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/WEB-SOLUTION-WITH-WORDPRESS/main/Images/createVolume2.png)
 <br>

We'll create **3 Volumes** in total. Once created we'll attachment them to the instance one-by-one
<br /> 
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/WEB-SOLUTION-WITH-WORDPRESS/main/Images/attachVolume.png)
 <br>

You should see a list of instances in the same availability zone as the ebs, im using project6
 <br /> 
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/WEB-SOLUTION-WITH-WORDPRESS/main/Images/attachVolume2.png)
 <br>