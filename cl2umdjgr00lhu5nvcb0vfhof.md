## Docker For Production Build

This is a complete hands-on tutorial for setting up an AWS EC2 instance for production using Docker, Portainer, Ngnix Reverse Proxy Manager & Cloudflare DNS. This is the first blog of the series further important topics such as VPC, Load Balancing etc... will be covered in a future blog post on this topic.  

Do first, understand later. That is the approach here. If you are new to some of these technologies, don’t worry about it. Just follow along, and dive into topics afterwards.

**In this guide, we will:**

- Setup a basic AWS EC2 Ubuntu 18.04 LTS/20.04 LTS instance.
- Setup Docker and Docker-Compose on Ubuntu instance.
- Running Portainer on Docker.
- Running Ngnix Reverse Proxy Manager on Docker.
- Setting up a domain on Cloudflare and setting up proxy settings.
- Access Portainer and Ngnix Reverse Proxy Manage using respective sub-domain.

**What you will need:**

- A AWS billing enabled account. (https://aws.amazon.com/)
- A Cloudflare account with a domain registered. (https://www.cloudflare.com/)

### What is Docker?

> Docker is an open-source project which provides the ability to create, package, and run applications in loosely isolated and contained environments called containers.

With all the isolation and security provided by the Docker platform, it allows you to run many containers simultaneously on a particular host.

### Reasons Why Docker Containers Are Widely Adopted Includes

- It allows developers to write code locally and share the work with their team using Containers.

- They can push their applications into the test environments, which are the containers and execute automated tests.

- When bugs are found, they can be fixed within the development environment and then redeploy.

- Getting a fix is as simple as pushing an updated image to the production environment.

### AWS EC2 Ubuntu Instance Setup

Head over to the official AWS website https://aws.amazon.com/

Check [AWS Official Docs](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/) for account setup. Kindly set up billing alerts so, that AWS doesn't charge you for using their services. 

![Screenshot 2022-05-06 at 11.30.53 AM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651816918494/q2Fu1RIK6.png align="left")

After the setup AWS console will open up. Please find the search bar at the top search for ec2. Select the first option. 

![Screenshot 2022-05-06 at 11.34.36 AM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651817093728/d67tqCWrl.png align="left")

Here, you will find all the details of ec2 instances running.

> **AWS Security Groups** help you secure your cloud environment by controlling how traffic will be allowed into your EC2 machines. With Security Groups, you can ensure that all the traffic that flows at the instance level is only through your established ports and protocols.

> An **Elastic IP** address is a static, public IPv4 address designed for dynamic cloud computing. You can associate an Elastic IP address with an instance or network interface in any VPC in your account.

Head over to instances(running) or instances option. At the top, you will find options such as Connect, Instance State, Actions and Launch Instance. Click on Launch Instance

![Screenshot 2022-05-06 at 1.30.22 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651824283232/pGLHp6LiW.png align="left")

In the above image, you can see that one instance is running. In your case, it will be empty as you haven't created any instance yet. 

Now, we have the list of all images we will be going with Ubuntu 20.04 LTS as most of them are aware of terminals commands and basic knowledge of how to work with Ubuntu. 

Ubuntu Server 20.04 LTS (HVM), SSD Volume Type - ami-05ba3a39a75be1ec4 (64-bit x86) / ami-075ebde7b27c12bc0 (64-bit Arm)

![Screenshot 2022-05-06 at 1.35.18 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651824335273/hJC0g1dm4.png align="left")

Now, the next steps would be configuring the instances as per our needs. For instance type select t2.micro (as it comes under the free tier). Instance configuration on default settings. For storage, the default is 8GB it could be scaled up to 30GB. Tag is optional this is required if you have set up a billing limit to monitor the costing and resources. 

![Screenshot 2022-05-06 at 1.46.23 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651825063978/2Cb_PP4D_.png align="left")

![Screenshot 2022-05-06 at 1.46.58 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651825085716/I6BU7jAbR.png align="left")

![Screenshot 2022-05-06 at 1.47.09 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651825104090/ejWSrtqXn.png align="left")

![Screenshot 2022-05-06 at 1.47.29 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651825121928/DguhbPzBK.png align="left")

The next step is the important step to configure security groups. By default port, 22 ie SSH will be enabled. Similarly, enable the following ports. 

- SSH: The Secure Shell Protocol is a cryptographic network protocol for operating network services securely over an unsecured network. (PORT **22**)

- HTTP: The Hypertext Transfer Protocol is an application layer protocol in the Internet protocol suite model for distributed, collaborative, hypermedia information systems. (PORT **80**)

- HTTPS: Hypertext Transfer Protocol Secure is an extension of the Hypertext Transfer Protocol. It is used for secure communication over a computer network and is widely used on the Internet. (PORT **443**)

- Portainer Control Panel: This requires a Custom TCP Port **9443**. 

- Ngnix Reverse Proxy Manager: This requires a Custom TCP Port of **81** to access its control panel. 

![Screenshot 2022-05-06 at 2.02.30 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651825960410/zrVZZHaEl.png align="left")

Source for HTTP, HTTPS must be **Anywhere** whereas SSH, TCP PORT 9443, and TCP PORT 81 could be changed to **My IP** or could be removed as well after Reverse Proxy is setup. 

After this click on Review and Launch to verify all the configurations are correct after which click on the Launch button. This will evoke a modal to create a Key Pair. 

This key is very important without this we can't access the instance via SSH. Choose to Create a new key pair option with Key pair type either RSA or ED25519 also enter a key pair name to identify the key pair. I'll be naming the key pair as **hashnode**.

![Screenshot 2022-05-06 at 2.09.47 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651826405927/R_Mm6olfz.png align="left")

Now, click on the download key pair button this will download a file with the name hashnode.cer or hashnode.pem . Later we will be using this key to access the instance using SSH. After this clicking on the launch instance will create an instance for us with the mentioned configurations. 

![Screenshot 2022-05-06 at 2.13.14 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651826611309/AYz1bxhRn.png align="left")

Now, head over to EC2 Console and click on the Elastic IPs option. In the top bar, you will find an option called Allocate Elastic IP address click on that then click on Allocate. Now, Amazon will allocate us an IPv4 address from Amazon's Pool. 

![Screenshot 2022-05-06 at 2.18.53 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651826942397/FCqfn7H9y.png align="left")

Now, you will find a top success bar button called Associate thus Elastic IP address click on that option. This will associate this elastic IP with our instance which we create some time ago. 

![Screenshot 2022-05-06 at 2.20.41 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651827052987/YhTPACE0H.png align="left")

Select the required instance id and Private IP address then tick mark if required on Allow this Elastic IP address to be associated. Then click on the Associate button to apply the necessary changes. 

Now, once the IP address is successfully associated head over to EC2 Instance Panel and you will be able to see the instance in running state. Now, we need to connect to this instance using SSH to get the required command to connect to the instance select the instance and click on Connect option which is on the top bar.

![Screenshot 2022-05-06 at 6.38.36 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651842526942/_l78n3dqo.png align="left")

Slide over to the SSH Client option wherein you will find the required command to connect to the instance. Follow the below-listed steps carefully

- Open an SSH client. (Eg Any terminal/ Putty)
- Locate your private key file. The key used to launch this instance is hashnode.pem/hashnode.cer
- Run this command, if necessary, to ensure your key is not publicly viewable.

```
chmod 400 hashnode.cer
``` 
- Connect to your instance using its Public DNS:

```
ec2-3-6-245-160.ap-south-1.compute.amazonaws.com
``` 
Example : 

```
ssh -i "hashnode.cer" ubuntu@ec2-3-6-245-160.ap-south-1.compute.amazonaws.com
``` 

Last step of this section we are supposed to connect to the instance using SSH. Now first, run the chmod command. Then using SSH establish a connection between your system and cloud instance.

> In Unix and Unix-like operating systems, **chmod** is the command and system call used to change the access permissions of file system objects sometimes known as modes.

![Screenshot 2022-05-06 at 6.52.22 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651843416647/tkVxDmRZH.png align="left")

![Screenshot 2022-05-06 at 6.54.19 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651843468620/ls-jF6SAL.png align="left")

Initially, when we run the command it will verify the signature. Type 'yes' to verify it then you will be successfully connected to the cloud instance or the ec2 instance. 

Now, let's update and upgrade all the Ubuntu Server 20.04 LTS. To do the same run the below listed commands one after the other. 


```
sudo apt-get update
``` 

```
sudo apt-get upgrade
``` 
Now, the cloud instance is successfully set up and ready for further steps.

### Install Docker

Check out the official docker documentation to install Docker on Ubuntu or blindly run the below commands one after the other. 

[Docker Documentation](https://docs.docker.com/engine/install/ubuntu/)

**Install using the Repository**

Before you install Docker Engine for the first time on a new host machine, you need to set up the Docker repository. Afterwards, you can install and update Docker from the repository.

**Set up the repository**

- Update the apt package index and install packages to allow apt to use a repository over HTTPS:

```
 sudo apt-get update

``` 
```
 sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
``` 
- Add Docker’s official GPG key:

```
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
``` 
- Use the following command to set up the stable repository. To add the nightly or test repository, add the word nightly or test (or both) after the word stable in the commands below.

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
``` 

### Install Docker Engine

- Update the apt package index, and install the latest version of Docker Engine, containers, and Docker Compose, or go to the next step to install a specific version:

```
sudo apt-get update
```

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

``` 
 
- Run the below command to check the version of docker installed this also verifies that docker has been successfully installed

```
docker --version
``` 

![Screenshot 2022-05-06 at 7.09.04 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651844353921/PwlK19LvY.png align="left")

Reference - [Docker Documentation](https://docs.docker.com/engine/install/ubuntu/)

### Install Portainer

[Portainer Offical Documentation](https://docs.portainer.io/v/ce-2.9/start/install/server/docker/linux)

- First, create the volume that Portainer Server will use to store its database:

```
sudo docker volume create portainer_data
``` 
- Then, download and install the Portainer Server container:

```
sudo docker run -d -p 9443:9443 --name portainer \
    --restart=always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data \
    portainer/portainer-ce:2.9.3
``` 

![Screenshot 2022-05-06 at 7.18.56 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651844950803/xcsDAC5-U.png align="left")

Now, Portainer is successfully set up and it's running on detach mode. To check the running containers run the below command. 

```
sudo docker ps
``` 
The output will be something like this

```
CONTAINER ID   IMAGE                          COMMAND        CREATED              STATUS              PORTS                                                           NAMES
11d36a8067a8   portainer/portainer-ce:2.9.3   "/portainer"   About a minute ago   Up About a minute   8000/tcp, 9000/tcp, 0.0.0.0:9443->9443/tcp, :::9443->9443/tcp   portainer

``` 
Now, Portainer is successfully running on **https://public-ip:9443**

![Screenshot 2022-05-06 at 7.25.36 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651845347241/zRl8WNT0y.png align="left")

If everything goes well then the above page where we need to set up our login has to show up. If this is not showing up check your security group configuration and also running docker containers.  Next click on **Get Started** then our environment will show up by default it will be having only one environment ie **local**.

![Screenshot 2022-05-06 at 7.29.34 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651845585960/B1yrve_TS.png align="left")

The next, step would be configuring Ngnix Reverse Proxy Manager and connecting domains to these applications. 

### Install Ngnix Reverse Proxy Manager

> Nginx Proxy Manager enables you to easily forward to your websites running at home or otherwise, including free SSL, without having to know too much about Nginx or Letsencrypt.

Refer [Nginix Reverse Proxy Manager Docs](https://github.com/jlesage/docker-nginx-proxy-manager)

- Launch the Nginx Proxy Manager docker container with the following command:

```
sudo docker run -d \
    --name=nginx-proxy-manager \
    -p 81:8181 \
    -p 80:8080 \
    -p 443:4443 \
    -v /docker/appdata/nginx-proxy-manager:/config:rw \
    jlesage/nginx-proxy-manager

``` 
- /docker/appdata/nginx-proxy-manager: This is where the application stores its configuration, log and any files needing persistency.

- Browse to http://public-ip:81 to access the Nginx Proxy Manager web interface.

![Screenshot 2022-05-06 at 7.36.57 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651846026434/A2Fy4kWgC.png align="left")

This shows that Ngnix Reverse Proxy Manager is successfully running.

![Screenshot 2022-05-06 at 7.39.09 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651846402763/P2Ll8JIcv.png align="left")

Default login credentials are 
- Email: admin@example.com
- Password: changeme

![Screenshot 2022-05-06 at 7.40.26 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651846590161/G6n0tC18T.png align="left")

After entering these credentials it will ask us to set new login credentials.  This completed the installation process next step is to link the domain using Cloudflare DNS. 

### Cloudflare DNS

[Cloudflare](https://dash.cloudflare.com/)

This is the most simple part. First, make sure you have a domain and you have added that domain to Cloudflare. [How to add a new domain to Cloudflare ?](https://community.cloudflare.com/t/step-1-adding-your-domain-to-cloudflare/64309)

![Screenshot 2022-05-06 at 7.54.08 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651847165209/UZRoqoqtA.png align="left")

First, create a Type A record for the name using **@** and for IPv4 use public-ip of the cloud instance also remember to disable the proxy status for now. 

![Screenshot 2022-05-06 at 7.58.34 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651847402945/Cj6u7VQEc.png align="left")

Now, create two CNAME records with names **p.@** and **n.@** and for Target use **@** also remember to disable proxy status for now. 


![Screenshot 2022-05-06 at 7.59.53 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651847406907/nZwx2c0l0.png align="left")

The next, step is to link the subdomain to Ngnix Reverse Proxy Manager. Head over to the proxy manager click on **Dashboard** then click on **Proxy Hosts**. Let's create a proxy host now

![Screenshot 2022-05-06 at 8.03.20 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651847662129/NGV86Qh3y.png align="left")

![Screenshot 2022-05-06 at 8.03.24 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651847663543/No39Q1W8CX.png align="left")

![Screenshot 2022-05-06 at 8.04.02 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651847674332/zuHfJf-0_.png align="left")

Refer to the above images and enter the required fields respectively. As Portainer runs on **HTTPS** scheme choose the correct scheme whereas Ngnix Reverse Proxy Manager runs on **HTTP**. In place of forwarding Hostname input to your local instance IP, you will find that in either in elastic IP configuration or instance configuration. 

Follow similar steps for Ngnix Reverse Proxy Manager

![Screenshot 2022-05-06 at 8.14.27 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651848284908/-GRIB3gpI.png align="left")

![Screenshot 2022-05-06 at 8.09.44 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651847997525/b1CQmipmK.png align="left")

![Screenshot 2022-05-06 at 8.09.25 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651847994875/N-_2Ax-K6.png align="left")

Now, we can enable proxy status in Cloudflare DNS also PORT 81, 9443 from Security Group. Also, Portianer & Ngnix Proxy Manager is now accessible from the sub-domain setup. 

![Screenshot 2022-05-06 at 8.10.46 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651848199679/ddmSGBIst.png align="left")

![Screenshot 2022-05-06 at 8.14.33 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651848292569/dHZNahvLi.png align="left")

Proxy is also working perfectly fine. The public-ip of the cloud instance is completely hidden thus providing a shield against DDOS attacks. 

![Screenshot 2022-05-06 at 8.17.17 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651848550708/3eVGqx_AS.png align="left")

Cheers,<br/>
Vignesh
