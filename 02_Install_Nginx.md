# Guide: Creating an EC2 Instance in Floci and Installing Nginx

This guide outlines the complete step-by-step process for provisioning a local EC2 instance using **Floci** and setting up **Nginx** inside an Amazon Linux 2 container environment.

---

## Prerequisites

- **Docker Desktop** installed and running.
- **Floci Engine & UI** running locally:
  - Floci Engine endpoint: `http://localhost:4566`
  - Floci UI: `http://localhost:4500`
- **AWS CLI** installed on your host machine.

---

## Step 1: Set Up Environment Variables

Before executing AWS CLI commands, configure your terminal to point to your local Floci environment instead of AWS production endpoints.

### PowerShell (Windows)
```powershell
$env:AWS_ENDPOINT_URL="http://localhost:4566"
$env:AWS_DEFAULT_REGION="us-east-1"
$env:AWS_ACCESS_KEY_ID="test"
$env:AWS_SECRET_ACCESS_KEY="test"
```

### Bash / Linux / macOS
```bash
export AWS_ENDPOINT_URL=http://localhost:4566
export AWS_DEFAULT_REGION=us-east-1
export AWS_ACCESS_KEY_ID=test
export AWS_SECRET_ACCESS_KEY=test
```

---

## Step 2: Provision the EC2 Instance

Choose one of two methods to launch your instance.

### Method A: Manual Creation (Interactive Installation)
Run the following AWS CLI command to spin up an instance:

```bash
aws ec2 run-instances \
    --image-id ami-0abcdef1234567890 \
    --instance-type t2.micro \
    --count 1
```

---

### Method B: Automated Creation (Using User Data Script)
Automate the Nginx installation during boot time by attaching a startup script.

1. Create a local file named `userdata.sh`:
   ```bash
   #!/bin/bash
   amazon-linux-extras install epel -y
   yum install -y nginx
   nginx
   ```

2. Launch the instance with the user-data flag:
   ```bash
   aws ec2 run-instances \
       --image-id ami-0abcdef1234567890 \
       --instance-type t2.micro \
       --count 1 \
       --user-data file://userdata.sh
   ```

---

## Step 3: Install Nginx Interactively (Method A Follow-up)

If you launched your instance using **Method A**, follow these steps to access the container directly and install Nginx.

### 1. Identify the Running Container
List all active Docker containers to find your EC2 container name:
```bash
docker ps
```
Look for a container named like `floci-aws-ec2-i-<instance_id>`.

### 2. Enter the Container Shell
Access the container's interactive shell:
```bash
docker exec -it <container_name_or_id> /bin/bash
```

### 3. Install Nginx on Amazon Linux 2
Execute the package manager commands inside the container:
```bash
# Enable the EPEL repository
amazon-linux-extras install epel -y

# Install Nginx
yum install -y nginx

# Start the Nginx daemon
nginx
```

---

## Step 4: Verification

1. Check if Nginx is actively running inside the container:
   ```bash
   ps aux | grep nginx
   ```
   Or send a local request inside the container:
   ```bash
   curl http://localhost
   ```

2. Open the **Floci UI** in your web browser at `http://localhost:4500` and navigate to **Compute** to view your running instance details.