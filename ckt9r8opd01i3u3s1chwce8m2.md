## Quick Guide To Deploy Using Docker

Do first, understand later. That is the approach here. If you are new to some of these technologies, don’t worry about it. Just follow along, and dive into topics afterwards.

**In this guide, we will:**

- Setup a basic React App.
- Create a React Docker Image.
- Setup an Azure Container Registry (ACR).
- Push the docker image to Azure Container Registry (ACR).
- Deploy using Azure Container Instance (ACI).
- Access the React app and set up Cloudflare SSL.

**What you will need:**

- Docker installed on your local system. (https://www.docker.com/products/docker-desktop)
- Node.js installed on your local machine. (https://nodejs.org/en/download/)
- Yarn installed on your local system. (https://yarnpkg.com/)
- An Microsoft Azure account. (sign up for free https://azure.microsoft.com/en-us/free/)
- A Cloudflare account with a domain registered. (https://www.cloudflare.com/)

### Creating a new React App

Head over to your local machine and open your Command Prompt (CMD) or PowerShell type 
```
npx create-react-app <app_name>
``` 
![1.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1630991574037/541pM6lyl.png)

To create a new react app. It installs all basic dependencies required to run a react app. **npx** will install and execute the create-react-app package. Make sure that your app name does not have any capital letters, else it will give an error. Once the command is executed successfully you will find instructions like (for eg let's take app name as react-app)

```
cd react-app

yarn start
``` 
![2.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1630991628754/eWPwmuYIK.png)

At this point, you can type the command cd <app name>, to go to your app folder. 

![3.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1630992143937/f-LDCqgJK.png)

Next, do yarn start, to start your app on the localhost by default react app will use port 3000.

![4.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1630992182856/OReaOrE1p.png)

If you have followed all the above-mentioned steps correctly you should see this page on your default browser. 

![5.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1630992342159/yqTKMUyLKn.png)

So, this will be our basic react app which we will be deploying using docker now open the current folder using your favorite text editor. If you don't have any text editor installed I recommended you to install [Vistall Studio Code](https://code.visualstudio.com/download). 

Now to exit the react app running on your terminal press CTRL + C. Then type 

```
code .
``` 
To open the current working folder on VS Code. To understand the folder structure and learn more about react app. I highly recommend you to check the official react documentation. [React](https://reactjs.org/docs/getting-started.html)

### Creating a React Docker Image

Once you have opened the current working folder with your favorite text editor. Create two new files named Dockerfile (Note that 'D' is in caps and this file doesn't have any extension) another one is .dockerignore (This file is the same as .gitignore file just to ignore 'node_modules' folder).

Now, your folder structure will look somewhat like this

![Screenshot 2021-09-07 111112.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630993288938/ISThSpM2d.png)

Now copy and paste the below lines to Dockerfile

```
#1. Base Image
FROM node:15.4 as build 

#2. Working Dir
WORKDIR /react-app

#3. Bring package.json file
COPY package*.json .

#5. Install all dependencies
RUN yarn install

#6. Copy files from the current working directory of the local system WORKDIR
COPY . .

#7. Get the optimized build of react app
RUN yarn run build

#8. Base Image
FROM nginx:1.19

#9. Get the ngnix configurations
COPY ./nginx/nginx.conf /etc/nginx/nginx.conf

#10. Copy the build folder of the react-app to ngnix HTML directory
COPY --from=build /react-app/build /usr/share/nginx/html

``` 

Now, let's configure the .dockerignorefile . Copy and paste the below snippet. 

```
**/node_modules
``` 

Now, you may think about where to find the Nginx configuration. So, now create a new folder named 'nginx'  and create a new file inside it and name it as 'nginx.conf'. The folder structure will look somewhat like

![Screenshot 2021-09-07 112747.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630994303502/tyjW4x7mv.png)

Copy and paste the below Nginx configurations in that file created in the last step. 

```
worker_processes  1;

events {
  worker_connections  1024;
}

http {
  server {
    listen 80;
    server_name  localhost;

    root   /usr/share/nginx/html;
    index  index.html index.htm;
    include /etc/nginx/mime.types;

    gzip on;
    gzip_min_length 1000;
    gzip_proxied expired no-cache no-store private auth;
    gzip_types text/plain text/css application/json application/javascript application/x-javascript text/xml application/xml application/xml+rss text/javascript;

    location / {
      try_files $uri $uri/ /index.html;
    }
  }
}
``` 

Now, the next setup is to create a docker image. To create a docker image docker must be installed on your local system. Make sure you do that first then run the below commands.

```
docker build -t react-app .
``` 
Here 'react-app' is the docker image name. Best practise to name a docker image is **docker_id/docker_image_name** . Here just to simplify things we are following the easy method.

![Screenshot 2021-09-07 114156.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630995131511/xYA7MFmHX.png)

Once the command is executed successfully. To check if the image is created you can run the docker command 

```
docker image ls 
``` 
If you have followed all the above-mentioned steps correctly you will find react-app in that list returned. 

![Screenshot 2021-09-07 114347.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630995366465/LfKD9wcCQ.png)

### Setup an Azure Container Registry (ACR)

First, search for [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/#:~:text=The%20Azure%20command%2Dline%20interface,with%20an%20emphasis%20on%20automation.) and install it for your OS. 

Once Azure CLI is installed successfully. To login using Azure CLI type 

```
az login
``` 
It will open a new browser tab enter your credentials and log in to your Azure account. On successful sign in it will return a success JSON to your terminal. Somewhat like this

![Screenshot 2021-09-07 115359.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630996106341/AZCa1Sif9.png)

Now, go to [Azure Portal](https://portal.azure.com/#home) Search for Container Registries

![Screenshot 2021-09-07 120055.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630996359907/g-U_b5Qdk.png)

Create a new Container Registry. Check the below image for the basic settings recommended.

![Screenshot 2021-09-07 120457.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630996553551/5n0mxHyUz.png)

Then click on 'Review+Create' to create a Container Registry. Once you get a success message saying validation passed click on the 'Create' button to create Container Registry. 

![Screenshot 2021-09-07 120813.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630997308570/sq6K5bSol.png)

Now, the next setup is to log in to this container registry created using Azure CLI. To read more check out the [Azure Docs](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-authentication?tabs=azure-cli).

```
az acr login --name reactappv
``` 

![Screenshot 2021-09-07 121100.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630996889777/60kL6fpzIc.png)

On success now we have to tag and push the 'react-app' docker image run the below docker command one by one. 

```
docker tag react-app reactappv.azurecr.io/react-app

``` 

```
docker push reactappv.azurecr.io/react-app
``` 
Finally, on push, you have to get a digest to verify the authenticity in the terminal. 

![Screenshot 2021-09-07 121603.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630997180995/WTOigb-6i.png)

Now, run the below command on the terminal to enable admin permission.

```
az acr update -n reactappv --admin-enabled true
``` 
On success, you will get a JSON response to your terminal. The next setup is to create a Container Instance and connect this Container Registry. 

### Setup an Azure Container Instance (ACI)

Now, go to [Azure Portal](https://portal.azure.com/#home) Search for Container Instances

![Screenshot 2021-09-07 122716.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630997845599/nl2SLvENY.png)

Now, click on the 'create' button to create a new Container Instance. Check the below images for recommended container instance settings. 

![Screenshot 2021-09-07 123046.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630998076560/Qzo-s0QXf.png)

![Screenshot 2021-09-07 123105.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630998078199/B46WGF9Hd.png)

Then click on 'Review+Create' to create a Container Instance. Once you get a success message saying validation passed click on the 'Create' button to create Container Instance. 

![Screenshot 2021-09-07 123234.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630998170214/bMN5WnaCw.png)

It may take a minute. Once your deployment is complete. Click on the 'go-to resource' button where you will find your public IP. 

![Screenshot 2021-09-07 123519.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630998440569/zFaYVTszK.png)

Copy and paste your public IP to your browser window. React template page should open if you have followed all the setup mentioned above correctly. 

![Screenshot 2021-09-07 123918.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630998573701/iBacUi0tF.png)

In the next part, we will be seeing how to set up free and secure SSL (Secure Sockets Layer) for any web application. Provided you have a domain configured at Cloudflare and a public IP. 

### Configure Cloudflare SSL

This is the most simple part. First, make sure you have a domain and you have added that domain to Cloudflare. [How to add a new domain to Cloudflare ?](https://community.cloudflare.com/t/step-1-adding-your-domain-to-cloudflare/64309)

In order to connect a domain to the public IP or the react app created in the previous section, you have to create a new 'a' record. 

[Cloudflare](https://dash.cloudflare.com/)

![Screenshot 2021-09-07 124946.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630999198758/PJtNIfl0w.png)

Now click on save. Hurray!! Now our react app is secure.

![Screenshot 2021-09-07 125048.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630999292721/rj-BouAxC.png)

Cheers,<br/>
Vignesh
