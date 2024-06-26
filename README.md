# aws_ec2-nodejs-nginx-clouldfare

How to deploy nodejs app to AWS EC2 Ubuntu 22 Server with free SSL and Nginx reverse proxy

## Installation instructions

### 1. Launch amazon ubuntu server in aws

1. **Sign in to AWS Console**:
   Open your web browser and navigate to the AWS Management Console. Sign in with your AWS account credentials.

2. **Navigate to EC2 Dashboard**:
   From the AWS Management Console, navigate to the EC2 dashboard by clicking on "Services" at the top of the page, and then selecting "EC2" under the "Compute" section.

3. **Launch Instance**:
   In the EC2 dashboard, click on the "Launch Instance" button to start the process of launching a new virtual server.

4. **Choose Amazon Machine Image (AMI)**:
   Choose an Ubuntu Server AMI from the list of available options. You can filter the list by selecting "Ubuntu" in the search bar.

5. **Choose Instance Type**:
   Select an instance type based on your requirements. The default options are usually suitable for basic usage, but you can customize as needed.

6. **Configure Instance Details**:
   Optionally, you can configure additional settings such as network, security groups, and IAM roles. For simplicity, you can stick with the default settings.

7. **Add Storage**:
   Configure the storage settings for your instance. You can adjust the size and type of the root volume as needed.

8. **Add Tags**:
   Optionally, you can add tags to your instance for easier management and organization.

9. **Configure Security Group**:
   Configure the security group settings to control inbound and outbound traffic to your instance. Make sure to allow SSH (port 22) access from your IP address.

10. **Review and Launch**:
    Review your instance configuration and click on the "Launch" button to launch your instance.

11. **Create Key Pair**:
    If you don't already have a key pair, create a new key pair and download the private key file (.pem) to your local machine. This key pair will be used to authenticate with your instance.

12. **Launch Instance**:
    Once you've downloaded the key pair, click on the "Launch Instances" button to launch your instance. 

### 2. Connect to ubuntu to install packages

1. **Find Public IP Address**:
   In the EC2 dashboard, find your newly launched instance and note down its public IP address.

2. **Connect via SSH**:
   Open a terminal on your local machine and use the `ssh` command to connect to your Ubuntu instance. Replace `your-ec2-public-ip` with the actual public IP address of your instance, and `your-key.pem` with the path to your private key file:

   ```bash
   ssh -i /path/to/your-key.pem ubuntu@your-ec2-public-ip

### 3. Install node and nvm

Install node version manager (nvm) by typing the following at the command line.

```bash
sudo su -
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash
```

Activate nvm by typing the following at the command line.

```bash
. ~/.nvm/nvm.sh
```

Use nvm to install the latest version of Node.js by typing the following at the command line.

```bash
nvm install node
```

Test that node and npm are installed and running correctly by typing the following at the terminal:

```bash
node -v
npm -v
```

### 4. Install git and Clone repository

#### 4.1 Install Git

Install Git: If Git is not already installed on your Ubuntu system, you can install it using the following command in your terminal:

```bash
sudo apt update
sudo apt install git
```

#### 4.2 Generate Personal Access Token

Visit the "Personal access tokens" settings page on GitHub: https://github.com/settings/tokens.
Click on "Generate new token" and enter your GitHub password to authenticate.
Give your token a descriptive name and select the scopes or permissions it requires. At a minimum, you'll need to select the repo scope to access repositories.
Once you've configured the token, click on "Generate token" to create it.
Copy the generated token to your clipboard. Note: This is the only time you will be able to see the token. Make sure to save it securely.

#### 4.3 Configure Git to Use the Token

```bash
git config --global credential.helper store
```

```bash
git credential-store --file ~/.git-credentials store <<EOF
protocol=https
host=github.com
username=<your-username>
password=<your-token>
EOF

```

#### 4.4 Clone repository

```sh
cd /home/ubuntu
```

```sh
git clone <yourRepository>
```

### 5. Run node app.js  (Make sure everything working)

```sh
cd <your-repository>
```

```sh
npm install
```

```sh
node app.js
```

### 8. Install Nginx web server

```sh
sudo apt install nginx
```

```sh
sudo nano /etc/nginx/sites-available/default
```

#### Add the following to the location part of the server block

```sh
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:<your-app-port>; #whatever port your app runs on
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
```

##### Check NGINX config

```sh
sudo nginx -t
```

##### Restart NGINX

```sh
sudo service nginx restart
```

### 9. Open Port of Node.js App in Security Group

#### Step 1: Sign in to AWS Console

1. Open your web browser and navigate to the [AWS Management Console](https://console.aws.amazon.com/).
2. Sign in with your AWS account credentials.

#### Step 2: Navigate to EC2 Dashboard

1. From the AWS Management Console, navigate to the EC2 dashboard by clicking on "Services" at the top of the page, and then selecting "EC2" under the "Compute" section.

#### Step 3: Select Security Group

1. In the EC2 dashboard, click on "Security Groups" in the navigation pane on the left.
2. Select the security group associated with your EC2 instance running the Node.js app.

#### Step 4: Edit Inbound Rules

1. Select the "Inbound Rules" tab for the selected security group.
2. Click on the "Edit inbound rules" button.

#### Step 5: Add Rule to Open Port

1. Click on the "Add Rule" button.
2. Choose the appropriate protocol (e.g., TCP) and port number for your Node.js app.
3. Specify the source IP or CIDR range that should be allowed to access the port (e.g., 0.0.0.0/0 to allow access from any IP address).
4. Click on the "Save rules" button to apply the changes.

#### Step 6: Verify Access

1. Once the inbound rule is added, verify that the port is open by attempting to access your Node.js app from the specified source IP or CIDR range.

#### Step 7: Monitor and Manage Security Group

1. You can monitor and manage your security group settings from the EC2 dashboard.
2. Make sure to review and update your security group rules regularly to ensure the security of your EC2 instances.

Congratulations! You have successfully opened the port of your Node.js app in the security group, allowing external access to your application.

### 10. Add Domain in Cloudflare to Get Free SSL

#### Step 1: Sign up/Login to Cloudflare

1. Open your web browser and navigate to the [Cloudflare website](https://www.cloudflare.com/).
2. Sign up for a new account or log in to your existing account.

#### Step 2: Add a Site to Cloudflare

1. After logging in, click on the "Add a Site" button on the Cloudflare dashboard.
2. Enter your domain name and click on the "Add Site" button.
3. Cloudflare will scan your domain's DNS records. Once the scan is complete, click on the "Next" button.

#### Step 3: Select a Plan

1. Cloudflare will prompt you to select a plan. Choose the "Free" plan and click on the "Confirm Plan" button.

#### Step 4: Update Your Name Servers

1. Cloudflare will provide you with two new name servers.
2. Update your domain's DNS settings at your domain registrar with the provided Cloudflare name servers.

#### Step 5: Enable Cloudflare Features

1. Once your DNS settings have propagated, Cloudflare will display a confirmation message.
2. You can then configure additional features such as SSL/TLS encryption, firewall rules, caching settings, etc.

#### Step 6: Enable SSL/TLS Encryption

1. Navigate to the "SSL/TLS" section of your Cloudflare dashboard.
2. Choose the SSL/TLS encryption mode you want to use (e.g., "Flexible", "Full", or "Full (Strict)").
3. Wait for Cloudflare to provision the SSL certificate for your domain.

#### Step 7: Verify SSL Certificate Installation

1. After Cloudflare has provisioned the SSL certificate, verify that SSL is enabled for your domain.
2. You can do this by visiting your domain in a web browser and checking for the padlock icon in the address bar.

#### Step 8: Monitor and Manage Your Domain

1. Once SSL is enabled, you can monitor and manage your domain settings from the Cloudflare dashboard.
2. Cloudflare provides various tools and analytics to help you optimize and secure your website.

Congratulations! You have successfully added your domain to Cloudflare and enabled free SSL encryption for your website.

### 11. Visit your website HTTPS://your-website

  Enjoy Your free Nodejs server with Free SSL :))))