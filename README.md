# Assignment-10
# MERN Stack Blog App Deployment on AWS

This document describes the steps to deploy a MERN stack blog application on AWS. The application consists of a React frontend, an Express.js backend, and a MongoDB Atlas database. The frontend is hosted on S3, and the backend runs on an EC2 instance. Media uploads are handled through a separate S3 bucket. [cite: 26, 27, 28, 1]

## Architecture

The architecture includes the following components:

* EC2 instance for the Express.js backend. [cite: 28, 4]
   
* S3 bucket for static website hosting of the React frontend and media storage. [cite: 28, 4]
   
* MongoDB Atlas for the database. [cite: 27, 4]
   
* IAM user for programmatic access to AWS resources. [cite: 28, 4]
   
* Security Group to control access to the EC2 instance. [cite: 28, 4]

**

## Deployment Steps

### Part 1: MongoDB Atlas Configuration (Optional)

* You can use the MongoDB connection string provided in the backend's `.env` file. [cite: 29, 30]
   
* Alternatively, you can set up your own MongoDB Atlas cluster:
    1.  Create a MongoDB Atlas account. [cite: 30, 31]
    2.  Set up a free cluster. [cite: 31]
    3.  Allow the EC2 instance's IP address in the network access settings. [cite: 31]
    4.  Create a database user and obtain the connection string. [cite: 31]

### Part 2: S3 Bucket for Frontend

1.  Create an S3 bucket named `yourname-blogapp-frontend` in the `eu-north-1` region. [cite: 31]
   
2.  Disable "Block all public access". [cite: 31]
   
3.  Enable static website hosting. [cite: 31]
   
4.  Add a bucket policy to allow public access:

    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": "*",
          "Action": "s3:GetObject",
          "Resource": "arn:aws:s3:::yourname-blogapp-frontend/*"
        }
      ]
    }
    ```
    [cite: 31, 32]

* **Deliverables:**
    * Screenshot of the S3 public URL loading `index.html`. [cite: 32]
    * `curl` output showing a `200 OK` response. [cite: 32, 7]

### Part 3: S3 for Media Uploads

1.  Create a second bucket named `yourname-blogapp-media`. [cite: 32]
   
2.  Disable "Block all public access". [cite: 32]
   
3.  Configure CORS for browser upload support:

    ```json
    [
      {
        "AllowedHeaders": ["*"],
        "AllowedMethods": ["GET", "PUT", "POST", "DELETE", "HEAD"],
        "AllowedOrigins": ["*"],
        "ExposeHeaders": ["ETag"]
      }
    ]
    ```
    [cite: 32]

4.  Test uploading and retrieving a file. [cite: 32]

* **Deliverables:**

    * Screenshot of file upload. [cite: 32, 7]

### Part 4: IAM User and Policy for S3 Media Bucket Access

1.  Go to IAM Console > Users > Add users. [cite: 32]
   
2.  Username: `blog-app-user`. [cite: 33]
   
3.  Permissions > Attach existing policies directly. [cite: 33]
   
4.  Create a custom policy with the following JSON (replace `yourname-blogapp-media` with your actual bucket name):

    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "s3:PutObject",
            "s3:GetObject",
            "s3:DeleteObject",
            "s3:ListBucket"
          ],
          "Resource": [
            "arn:aws:s3:::yourname-blogapp-media",
            "arn:aws:s3:::yourname-blogapp-media/*"
          ]
        }
      ]
    }
    ```
    [cite: 33]

5.  Attach this policy to the user. [cite: 33, 8]
   
6.  \*Important\*: Create and save the Access Key ID and Secret Access Key for this user. [cite: 34, 9]
   
7.  You will not be able to view the secret key again. [cite: 35, 10]
   
8.  And DO NOT share these credentials in your submission or the solution GitHub repo! [cite: 36, 11, 19]

### Part 5: EC2 Backend Setup

1.  Launch a `t3.micro` instance in `eu-north-1` using Ubuntu 22.04 LTS. [cite: 37, 12]
   
2.  Allow incoming traffic on necessary ports: SSH (22), HTTP (80), HTTPS (443), and Custom TCP (5000). [cite: 37, 12]

    * Add this Inbound rule:

        | Type       | Protocol | Port Range | Source    | Description        |
        | :--------- | :------- | :--------- | :-------- | :----------------- |
        | Custom TCP | TCP      | 5000       | 0.0.0.0/0 | Allow public traffic |
        [cite: 38, 13]

3.  SSH into the instance and run the User Data script:

    ```bash
    #!/bin/bash
    apt update -y
    apt install -y git curl unzip tar gcc g++ make unzip
    
    su ubuntu << 'EOF'
    export NVM_DIR="$HOME/.nvm"
    curl -o- [https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh](https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh) | bash
    source "<span class="math-inline">NVM\_DIR/nvm\.sh"
nvm install \-\-lts
nvm use \-\-lts
npm install \-g pm2
EOF
\# MongoDB Shell
curl \-L \[https\://downloads\.mongodb\.com/compass/mongosh\-2\.1\.1\-linux\-x64\.tgz\]\(https\://downloads\.mongodb\.com/compass/mongosh\-2\.1\.1\-linux\-x64\.tgz\) \-o mongosh\.tgz
tar \-xvzf mongosh\.tgz
mv mongosh\-\*/bin/mongosh /usr/local/bin/mongosh
chmod \+x /usr/local/bin/mongosh
rm \-rf mongosh\*
\# AWS CLI
curl "\[https\://awscli\.amazonaws\.com/awscli\-exe\-linux\-x86\_64\.zip\]\(https\://awscli\.amazonaws\.com/awscli\-exe\-linux\-x86\_64\.zip\)" \-o "awscliv2\.zip"
unzip awscliv2\.zip
\./aws/install
rm \-rf aws awscliv2\.zip
\# Clone the MERN app repository
git clone \[https\://github\.com/cw\-barry/blog\-app\-MERN\.git\]\(https\://github\.com/cw\-barry/blog\-app\-MERN\.git\) /home/ubuntu/blog\-app
\# Navigate to the backend directory
cd /home/ubuntu/blog\-app/backend
\# Create \.env file
cat \> \.env << EOF
PORT\=5000
HOST\=0\.0\.0\.0
MODE\=production
\# Database configuration
\# if you have your own MongoDB connection string you can use it here
MONGODB\=mongodb\+srv\://test\:qazqwe123@mongodb\.txkjsso\.mongodb\.net/blog\-app
\# JWT Authentication
JWT\_SECRET\=</span>(openssl rand -hex 32)
    JWT_EXPIRE=30min
    JWT_REFRESH=$(openssl rand -hex 32)
    JWT_REFRESH_EXPIRE=3d
    
    # AWS S3 Configuration
    AWS_ACCESS_KEY_ID=<access-key-from-step-6>
    AWS_SECRET_ACCESS_KEY=<secret-key-from-step-6>
    AWS_REGION=eu-north-1
    S3_BUCKET=<your-s3-media-bucket-name>
    MEDIA_BASE_URL=https://<your-s3-media-bucket-name>.eu-north-1.amazonaws.com
    
    # Misc
    DEFAULT_PAGINATION=20
    EOF
    
    # Navigate to the frontend directory
    cd ../frontend
    
    # Create .env file
    cat > .env << EOF
    VITE_BASE_URL=http://<your-ec2-dns>:5000/api
    VITE_MEDIA_BASE_URL=https://<your-s3-media-bucket-name>.eu-north-1.amazonaws.com
    EOF
    
    # Configure AWS CLI
    aws configure
    # (Enter your AWS Access Key ID and Secret Access Key when prompted)
    # Set default region to eu-north-1
    # Set default output format to json
    
    # Deploy Backend
    cd /home/ubuntu/blog-app/backend
    npm install
    mkdir -p logs
    pm2 start index.js --name "blog-backend"
    pm2 startup
    sudo pm2 startup systemd -u ubuntu
    sudo env PATH=$PATH:/home/ubuntu/.nvm/versions/node/v22.15.0/bin /home/ubuntu/.nvm/versions/node/v22.15.0/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu-hp/home/ubuntu
    pm2 save
    
    # Build and Deploy Frontend
    cd /home/ubuntu/blog-app/frontend
    npm install -g pnpm@latest-10
    pnpm install
    pnpm run build
    aws s3 sync dist/ s3://<your-s3-frontend-bucket-name>/
    ```
    [cite: 40, 42, 43, 15, 17, 18, 19, 20]

* **Deliverables:**

    * Screenshot of the backend server running via pm2. [cite: 20]
    * Screenshot of the frontend S3 web page. [cite: 20, 7]

## Success Criteria

The deployment is considered successful if:

1.  The application runs with full MERN functionality (create blog posts, upload media). [cite: 20]
   
2.  The frontend and media files are publicly accessible via S3. [cite: 20]
   
3.  Backend access is secured using environment variables. [cite: 20]
   
4.  MongoDB Atlas is reachable and stores blog data correctly. [cite: 20]

## Cleanup

* Stop the EC2 instance.
* Remove the IAM credentials.
* Delete the S3 buckets if they are no longer needed.
