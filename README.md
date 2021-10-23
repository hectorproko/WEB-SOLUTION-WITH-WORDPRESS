# WEB-SOLUTION-WITH-WORDPRESS
In this project we will prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress. 

put what you are doing as far as partitions and logical volumes etc

In this project we will prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress. This will be implemented based on what is called the **Three-tier Architecture**.

In this project:
1. Presentation Layer (PL): Any Browser
2. Business Layer (BL): An EC2 Linux Server as a web server (where WordPress is installed)
3. Data Access or Management Layer (DAL): An EC2 Linux server as a database (DB) server (MySQL)

We will create **EBS Volumes** partition them, create **Volume Groups**, create **Logical Volumes** format and mount them.

Technologies/Tools used:
* AWS (EC2)(EBS)
* Red Hat Enterprise Linux 8 (HVM)
* MySQL
* Wordpress
* GitBash
