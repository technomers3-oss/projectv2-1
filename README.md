# ☁️ Cloud Computing Lab Manual

> AWS Lab Experiments: EC2, EBS, EFS, and S3

---

## Table of Contents

- [Lab 1 — EC2 Instance Launch & Connect](#lab-1--ec2-instance-launch--connect)
- [Lab 2 — EBS Volume Management](#lab-2--ebs-volume-management)
- [Lab 3 — EFS (Elastic File System)](#lab-3--efs-elastic-file-system)
- [Lab 4 — Amazon S3](#lab-4--amazon-s3)

---

## Lab 1 — EC2 Instance Launch & Connect

### Part A — Launch Linux EC2 Instance

**Step 1:** Login to AWS Console / AWS Academy Learner Lab.

**Step 2:** Choose Region:
- Asia Pacific (Mumbai), **or**
- N. Virginia

**Step 3:** Navigate to `Services → EC2 → Launch Instance`

**Step 4:** Configure Instance with the following settings:

| Setting | Value |
|---|---|
| Name | `My_VS1` |
| AMI | Ubuntu / Amazon Linux |
| Architecture | 64-bit (x86) |
| Instance Type | `t3.micro` |
| Key Pair Type | RSA |
| Key Pair Format | `.pem` |

> Keep default network and storage settings, then click **Launch Instance**.

---

### Part B — Connect Using SSH

**Step 1:** Open Terminal / PowerShell / Git Bash.

**Step 2:** Navigate to your key file location:

```bash
cd Downloads
```

**Step 3:** Set correct permissions on the key file:

```bash
chmod 400 mykey.pem
```

**Step 4:** Connect to your EC2 instance:

For **Ubuntu** AMI:
```bash
ssh -i mykey.pem ubuntu@<Public-IP>
```

For **Amazon Linux** AMI:
```bash
ssh -i mykey.pem ec2-user@<Public-IP>
```

✅ **Connected Successfully**

---

### Part C — Connect Using PuTTY

**Step 1:** Install **PuTTY** and **PuTTYgen**.

**Step 2:** Convert `.pem` to `.ppk`:
1. Open **PuTTYgen**
2. Click **Load** → Select your `.pem` file
3. Click **Save Private Key** → Save as `keyname.ppk`

**Step 3:** Open **PuTTY** and enter the Host Name:

```
ubuntu@<Public-IP>
```
or
```
ec2-user@<Public-IP>
```

**Step 4:** Attach the `.ppk` key by navigating to:

```
Connection → SSH → Auth → Credentials
```

Browse and select the `.ppk` file.

**Step 5:** Click **Open**.

✅ **Connected Successfully**

---

### Part D — Connect Using AWS CloudShell

**Step 1:** Open **CloudShell** in the AWS Console.

**Step 2:** Upload your `.pem` file.

**Step 3:** Set key permissions:

```bash
chmod 400 mykey.pem
```

**Step 4:** Connect:

```bash
ssh -i mykey.pem ubuntu@<Public-IP>
```
or
```bash
ssh -i mykey.pem ec2-user@<Public-IP>
```

✅ **Connected Successfully**

---

### Part E — Launch Windows EC2 and Connect via RDP

**Step 1:** Launch a Windows EC2 Instance:
- Select **Windows Server AMI**
- Instance Type: `t2.micro`
- Allow **RDP Port 3389**
- Download the `.pem` key

**Step 2:** Retrieve the Administrator Password:

```
EC2 → Connect → RDP Client → Get Password
```

Upload your `.pem` file and decrypt the password.

**Step 3:** Open Remote Desktop:

```
mstsc
```

**Step 4:** Enter the **Public IP / DNS** and login with:

| Field | Value |
|---|---|
| Username | `Administrator` |
| Password | *(Decrypted password from Step 2)* |

✅ **Connected Successfully**

---

## Lab 2 — EBS Volume Management

> **Workflow Summary:**
> `Create Volume → Attach → lsblk → fdisk → mkfs → mkdir → mount → fstab → reboot`

---

### Step 4 — Verify Attached Disk

```bash
lsblk
```

Expected output:
```
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
nvme0n1     ...               # Root Volume
nvme1n1     ...               # New EBS Disk
```

---

### Step 5 — Create Partition

```bash
sudo fdisk /dev/nvme1n1
```

Inside `fdisk`, enter the following commands in sequence:

```
n   # New partition
p   # Primary partition
1   # Partition number 1
    # (press Enter for default first sector)
    # (press Enter for default last sector)
w   # Write and save changes
```

---

### Step 6 — Verify Partition

```bash
lsblk
```

Expected output:
```
nvme1n1
└─nvme1n1p1
```

---

### Step 7 — Format Partition with XFS

```bash
sudo mkfs.xfs /dev/nvme1n1p1
```

Verify the filesystem:
```bash
lsblk -fs
```

Expected: `nvme1n1p1` shows type `xfs`

---

### Step 8 — Create Mount Directory

```bash
sudo mkdir /mnt/cseb
```

---

### Step 9 — Mount the Volume

```bash
sudo mount /dev/nvme1n1p1 /mnt/cseb
```

Verify:
```bash
df -h
```

Expected output:
```
Filesystem       Size  Used Avail  Mounted on
/dev/nvme1n1p1   100G  ...  ...   /mnt/cseb
```

---

### Step 10 — Make Mount Persistent (fstab)

Get the UUID of the partition:
```bash
sudo blkid /dev/nvme1n1p1
```

Example output:
```
/dev/nvme1n1p1: UUID="e14d4020-d4f3-4f88-bd4c-89aa97aca41a" TYPE="xfs"
```

Edit `/etc/fstab`:
```bash
sudo nano /etc/fstab
```

Add the following line at the end (replace UUID with your own):
```
UUID=e14d4020-d4f3-4f88-bd4c-89aa97aca41a  /mnt/cseb  xfs  defaults,nofail  0  0
```

Save and exit:
```
CTRL + O  →  ENTER  →  CTRL + X
```

Test the fstab configuration:
```bash
sudo mount -a
```

> If no error appears, the configuration is correct.

---

### Step 11 — Create and Verify a Test File

```bash
cd /mnt/cseb
sudo touch testfile.txt
echo "EBS Volume Persistence Test" | sudo tee testfile.txt
cat testfile.txt
```

Expected output:
```
EBS Volume Persistence Test
```

---

### Step 12 — Unmount Before Detaching

```bash
sudo umount /mnt/cseb
df -h
```

---

### Detach and Reattach Volume

**Detach from AWS Console:**
```
EC2 Dashboard → Volumes → Select Volume → Actions → Detach Volume
```
Wait until state shows `available`.

**Attach to another EC2 instance:**
```
Select Volume → Actions → Attach Volume → Select Instance → Device: /dev/sdf
```
Wait until state shows `in-use`.

---

### On the Second EC2 Instance

Check attached disk:
```bash
lsblk
```

Create a new mount directory:
```bash
sudo mkdir /mnt/archana
```

Mount the existing EBS volume:
```bash
sudo mount /dev/nvme1n1p1 /mnt/archana
```

Verify data persistence:
```bash
cd /mnt/archana
ls
cat testfile.txt
```

Expected output:
```
EBS Volume Persistence Test
```

✅ Data persisted across EC2 instances successfully.

---

## Lab 3 — EFS (Elastic File System)

### Step 1 — Launch EC2 Instances

Create **2 EC2 Instances** with the following configuration:

| Setting | EFS-1 | EFS-2 |
|---|---|---|
| AMI | Amazon Linux | Amazon Linux |
| Instance Type | `t2.micro` | `t2.micro` |
| Key Pair | `kgf` | `kgf` |
| Subnet | `us-east-1a` | `us-east-1c` |

**Security Group Rules:**

| Type | Port | Source |
|---|---|---|
| SSH | 22 | Anywhere |
| NFS | 2049 | Anywhere |

---

### Step 2 — Create EFS File System

1. Go to **EFS** in the AWS Console
2. Click **Create File System**
3. Select the **Default VPC**
4. In **Network Settings**: Remove the default SG, add the NFS Security Group
5. Click **Create File System**

---

### Step 3 — Connect to EC2 Instances

SSH into each instance:
```bash
ssh -i kgf.pem ec2-user@<Public-IP>
```

Switch to root:
```bash
sudo su
```

Create the EFS mount directory:
```bash
mkdir efs
```

Install EFS utilities:
```bash
yum install -y amazon-efs-utils
```

---

### Step 4 — Attach EFS to EC2 Instances

Mount the EFS (replace with your EFS ID):
```bash
sudo mount -t efs -o tls fs-04ec940b87616695f:/ efs
```

---

### Step 5 — Test EFS

```bash
cd efs
touch fs
ls
```

Expected output:
```
fs
```

---

### Step 6 — Verify File Sharing

On the **second EC2 instance**:
```bash
cd efs
ls
```

Expected output:
```
fs
```

✅ EFS is successfully mounted and file sharing works between both EC2 instances.

---

## Lab 4 — Amazon S3

### Part 1 — S3 Bucket Creation

#### Step 1 — Create Bucket

1. Go to **S3** in the AWS Console → Click **Create Bucket**
2. Configure:

| Setting | Value |
|---|---|
| Bucket Name | `salaar-public-bucket` |
| Region | N. Virginia |
| ACLs | Enabled |
| Block All Public Access | **Unchecked** |
| Bucket Versioning | Enabled |

3. Click **Create Bucket**

---

#### Step 2 — Enable Public Access (ACL)

```
Open Bucket → Permissions Tab → Access Control List (ACL) → Edit
→ Everyone (public access): ✅ Read → Save Changes
```

---

#### Step 3 — Upload File

```
Open Bucket → Upload → Select File
→ Permissions: ACL Read Access Enabled → Upload
```

---

#### Step 4 — Verify Versioning

Upload the same file again and go to **Versions** — different versions of the file will be displayed.

---

### Part 2 — S3 Replication

#### Step 1 — Create Destination Bucket

In a **different AWS Region**, create another bucket with:
- ACL Enabled
- Block Public Access: **Unchecked**
- Versioning: Enabled

#### Step 2 — Create Replication Rule

```
Source Bucket → Management Tab → Replication Rules → Create Rule
```

| Setting | Value |
|---|---|
| Scope | Apply to all objects |
| Destination | *(Select destination bucket)* |
| IAM Role | `LabRole` |

Click **Save**.

#### Step 3 — Test Replication

Upload a PDF to the **Source Bucket** → open the **Destination Bucket**.

✅ The file automatically appears in the destination bucket.

---

### Part 3 — Static Website Hosting

#### Step 1 — Create Static Website Bucket

| Setting | Value |
|---|---|
| Bucket Name | `my-static-site-bucket-123` |
| ACLs | Enabled |
| Block All Public Access | **Unchecked** |

#### Step 2 — Upload Website Files

Upload both:
- `index.html`
- `error.html`

#### Step 3 — Enable Static Website Hosting

```
Bucket → Properties Tab → Static Website Hosting → Edit → Enable
```

| Field | Value |
|---|---|
| Index Document | `index.html` |
| Error Document | `error.html` |

#### Step 4 — Enable Public Access (Bucket + Object Level)

For the **bucket**:
```
Permissions → ACL → Edit → Everyone (public access): ✅ Read → Save
```

Repeat the same for **each object** (`index.html`, `error.html`):
```
Open Object → Permissions → ACL → Edit → Enable Public Read → Save
```

#### Step 5 — Access the Website

```
Properties → Static Website Hosting → Copy Website Endpoint
```

Example URL:
```
http://my-static-site-bucket-123.s3-website-us-east-1.amazonaws.com
```

Paste in browser → `index.html` opens successfully.

#### Step 6 — Test Error Page

Open an invalid URL:
```
http://<bucket-name>.s3-website-<region>.amazonaws.com/invalid.html
```

✅ `error.html` is displayed correctly.

---

### Website Files

#### `index.html`

```html
<!DOCTYPE html>
<html>
<head>
  <title>My Static Website</title>
  <style>
    body {
      background-color: #f2f2f2;
      font-family: Arial;
      text-align: center;
      padding-top: 100px;
    }
    h1 { color: #2c3e50; font-size: 45px; }
    p  { color: #555; font-size: 22px; }
    .box {
      background: white;
      width: 60%;
      margin: auto;
      padding: 40px;
      border-radius: 10px;
      box-shadow: 0px 0px 10px gray;
    }
  </style>
</head>
<body>
  <div class="box">
    <h1>Welcome to My AWS S3 Website</h1>
    <p>Static Website Hosting using Amazon S3</p>
    <p>Cloud Computing Lab Experiment</p>
  </div>
</body>
</html>
```

---

#### `error.html`

```html
<!DOCTYPE html>
<html>
<head>
  <title>Error Page</title>
  <style>
    body {
      background-color: #ffe6e6;
      font-family: Arial;
      text-align: center;
      padding-top: 120px;
    }
    h1 { color: red; font-size: 50px; }
    p  { font-size: 22px; color: #333; }
    a  { text-decoration: none; color: blue; font-size: 20px; }
  </style>
</head>
<body>
  <h1>404 Error</h1>
  <p>Oops! The page you are looking for does not exist.</p>
  <a href="index.html">Back to Home</a>
</body>
</html>
```

---

*Cloud Computing Lab Manual — AWS EC2 · EBS · EFS · S3*

check your region is us east 1 /2 go to your aws cloud shell then run : aws s3 cp s3://cc-lab-share-79050/cc_lab.zip . && unzip -o cc_lab.zip && chmod +x scripts/.sh cleanup/.sh && ls scripts/ cleanup/ .

to run experiment : ls cd scripts bash expname.sh

it will automatically run the script and then create your experiment

for sns sqs run : bash expname.sh youremail@gmail.com

in order to clear all the setup of experiment : cd .. cd cleanup run : bash expname.sh

s3 dynamodb integration code 

```python
import json
import boto3
from uuid import uuid4

def lambda_handler(event, context):
    s3 = boto3.client("s3")
    dynamodb = boto3.resource('dynamodb')
    dynamoTable = dynamodb.Table('nabutta-db')

    print("Event:", event)

    # Check if the 'Records' key is present in the event
    if 'Records' in event:
        for record in event['Records']:
            bucket_name = record['s3']['bucket']['name']
            object_key = record['s3']['object']['key']
            size = record['s3']['object'].get('size', -1)
            event_name = record.get('eventName', 'Unknown')
            event_time = record.get('eventTime', 'Unknown')
            
            dynamoTable.put_item(
                Item={
                    'id': str(uuid4()), 
                    'Bucket': bucket_name, 
                    'Object': object_key,
                    'Size': size, 
                    'Event': event_name, 
                    'EventTime': event_time
                }
            )
        return {"status": "success"}
    else:
        print("No 'Records' key found in the event.")
        return {"status": "no_records"}

```
CDN

[https://cdn.jsdelivr.net/gh/technomers3-oss/technomersproject@main/WEEK-7-LAMBDA.pdf](https://cdn.jsdelivr.net/gh/technomers3-oss/technomersproject@main/WEEK-7-LAMBDA.pdf)

[https://cdn.jsdelivr.net/gh/technomers3-oss/technomersproject@main/week-8.pdf](https://cdn.jsdelivr.net/gh/technomers3-oss/technomersproject@main/week-8.pdf)


[https://cdn.jsdelivr.net/gh/technomers3-oss/technomersproject@main/week9.pdf](
https://cdn.jsdelivr.net/gh/technomers3-oss/technomersproject@main/week9.pdf)

[https://cdn.jsdelivr.net/gh/technomers3-oss/technomersproject@main/WEEK-10.pdf](https://cdn.jsdelivr.net/gh/technomers3-oss/technomersproject@main/WEEK-10%20.pdf)

[https://cdn.jsdelivr.net/gh/technomers3-oss/technomersproject@main/Experiment-11.pdf](https://cdn.jsdelivr.net/gh/technomers3-oss/technomersproject@main/Experiment-11.pdf)

[https://cdn.jsdelivr.net/gh/technomers3-oss/technomersproject@main/week-12.pdf](https://cdn.jsdelivr.net/gh/technomers3-oss/technomersproject@main/week-12.pdf)

[https://cdn.jsdelivr.net/gh/technomers3-oss/technomersproject@main/week-13.pdf](https://cdn.jsdelivr.net/gh/technomers3-oss/technomersproject@main/week-13.pdf)

[https://scam-pied.vercel.app/](https://scam-pied.vercel.app/)
