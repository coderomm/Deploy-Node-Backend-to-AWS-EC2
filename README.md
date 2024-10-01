This is a full guide of how I learned and deployed on aws.

---

AWS is Amazon's cloud service that lets you:

1. Rent servers
2. Manage domains
3. Upload files (like mp4s, jpgs, mp3s)
4. Autoscale servers
5. Create Kubernetes clusters

---

# **EC2 Servers**

Virtual machines (VMs) on AWS are called `EC2 servers`.

EC2 stands for "Elastic Compute Version 2."

- **Elastic**: You can increase or decrease the size of the machine.
- **Compute**: It's a machine.

---

## **Step 1 - Create a New EC2 Server**

1. Click on `Launch a new instance`.
2. Give your instance a name.
3. Select an OS (e.g., Ubuntu).
4. Choose a free instance type.
5. Create a new key pair (e.g., `youlayer.pem`).
6. Configure storage.
7. Allow traffic on HTTP/HTTPS/SSH.

---

## **Step 2 - SSH into Your Server**

Go to the folder where your `.pem` file is stored, and run this command in your terminal:

```bash
ssh -i "youlayer2.pem" ubuntu@<Your_Server_IP>
```

If you're connecting again but with a different `.pem` file, you might get an error like:

```
The authenticity of host '13.235.103.56 (13.235.103.56)' can't be established.
ECDSA key fingerprint is SHA256:c9diw84yQzQve33YTJMLvV7ABFZpwf6O+5te77z9Ht0.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
Host key verification failed.
```

To fix this, remove the old entry from your `known_hosts` file:

```bash
C:\\Users\\YourUsername\\.ssh\\known_hosts
```

Then, run the SSH command again, adding an option to skip key verification:

```bash
ssh -i "youlayer2.pem" ubuntu@<Your_Server_IP> -o StrictHostKeyChecking=no
```

You might also get an error about file permissions:

```
WARNING: UNPROTECTED PRIVATE KEY FILE!
Permissions for 'youlayer2.pem' are too open.

Detailed Error : 
Warning: Permanently added '13.235.103.56' (ECDSA) to the list of known hosts.
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions for 'youlayer2.pem' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "youlayer2.pem": bad permissions
[ubuntu@13.235.103.5](mailto:ubuntu@13.235.103.57)6: Permission denied (publickey).
```

To fix this on Windows:

1. Right-click on the `.pem` file and select `Properties`.
2. Go to `Security > Advanced > Disable inheritance`.
3. Remove all users.
4. Add your Windows username and give it full permissions.

After this, run the SSH command again, and you should be logged in.

Reference - https://stackoverflow.com/questions/11380955/chmod-not-recognized-as-internal-or-external-command/74359284#74359284

Then, run the SSH command again, adding an option to skip key verification:

```bash
ssh -i "youlayer2.pem" ubuntu@<Your_Server_IP>
```

Then Clone repo

```jsx
git clone https://github.com/coderomm/youlayer-tech
```

---

Here’s how I handled the copy-paste issue while using SSH:

1. **Windows Command Prompt or PowerShell**:
    
    To paste in the terminal:
    
    - Right-click anywhere inside the terminal window to paste.
    
    To enable keyboard shortcuts:
    
    - Right-click on the title bar of the Command Prompt or PowerShell window.
    - Select `Properties`.
    - Check the option `Use Ctrl+Shift+C/V as Copy/Paste`.
    
    Now, I can paste using `Ctrl + Shift + V` instead of just `Ctrl + V`.
    

---

## **Step 3 - Setup Node.js on Your EC2 Server**

1. Install Node.js and npm.💡

https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-20-04

### **Install Node.js using NodeSource PPA:**

This method installs the latest stable version of Node.js.

1. **Update your package list**:
    
    ```bash
    sudo apt update
    ```
    
2. **Install prerequisites**:
    
    ```bash
    sudo apt install -y curl
    ```
    
3. **Install Node.js**:
    
    ```bash
    sudo apt install -y nodejs
    ```
    
4. **Verify Node.js and npm installation**:
    
    ```bash
    node -v
    npm -v
    ```
    

This will install the latest long-term support (LTS) version of Node.js along with `npm` (Node Package Manager).

1. Go to your backend directory and run:

```bash
npm install
```

1. Create a `.env` file to store your environment variables (like `FIREBASE_PRIVATE_KEY`). Use the following command:

```bash
touch .env
nano .env
```

Add your variables, save, and exit the editor by:

- Pressing `Ctrl + O` to save.
- Press `Ctrl + X` to exit.

---

## **Step 4 - Run Your Backend**

After setting up the `.env` file, run your backend:

```bash
node app.js
```

You should see something like this:

```
Server is running on localhost:3000
Successfully connected to MongoDB Atlas!
```

That's it! You're now running your app on an EC2 server.

---

## Step 5 - Step-by-Step Guide

To create a subdomain in Hostinger and point it to my EC2 instance on AWS, I followed these steps:

I needed to point my subdomain `backend.youlayer.tech` to my AWS EC2 instance while keeping the main domain (`youlayer.tech`) pointed to Vercel for the frontend. Here’s how I did it:

1. **Added A Record for the Subdomain:**
    - I went into the DNS Zone Editor in Hostinger.
    - Then, I added a new A record:
        - **Type**: A
        - **Name**: `backend` (so it points to (`backend.youlayer.tech`)
        - **Target**: I used my EC2 IP (`13.235.103.57`).
        - **TTL**: I left it at 14400 (or you can use 3600 for faster propagation).
    - This didn’t interfere with my existing A record for `youlayer.tech`, which points to Vercel.
2. **Set Up Security Group in AWS:**
    - I made sure the security group for my EC2 instance allowed HTTP (port 80) and HTTPS (port 443) traffic.
3. **DNS Propagation:**
    - I waited for DNS propagation (it can take a few minutes to hours). I used `whatsmydns.net` to check the status of `backend.youlayer.tech`.
4. **Accessed the Subdomain:**
    - Once the DNS changes propagated, I was able to access my backend via `backend.youlayer.tech`.

This way, my frontend stays on Vercel and my backend runs on the EC2 instance.

---

## Step 6 - Nginx

### **Here’s how I handled installing `pm2` globally:**

I use `pm2` to keep my Node.js app running continuously, even after crashes or reboots. It also helps with monitoring, log management, and scaling for better performance.

Since I was installing `pm2` globally and it required writing to a protected system directory, I needed to use elevated permissions. So, I ran the command with `sudo`:

```bash
sudo apt update
sudo apt install nginx
```

This gave the necessary permissions to install `pm2` globally without any issues.

---

## **Step 7 - What is a reverse proxy?**

### **Create reverse proxy**

```jsx
sudo rm sudo vi /etc/nginx/nginx.conf
sudo vi /etc/nginx/nginx.conf
```

```jsx
events {
    # Event directives...
}

http {
	server {
    listen 80;
    server_name backend.youlayer.tech;

    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
	}
}
```

### **How to edit/save a file through Ubuntu Terminal**

> Press Esc and then type below respectively
> 
> 
> ```
> :wq //save and exit
> :q! //exit without saving
> ```
> 
1. sudo nano path_to_file/file_name

### **Start the Backend server**

```jsx
node index.js
```

---

## Step 8 - Add SSL Certificate

Use https://certbot.eff.org/