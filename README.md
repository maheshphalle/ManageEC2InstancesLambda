# ManageEC2InstancesLambda

# AWS Lambda EC2 Management

## **Objective**
This AWS Lambda function automatically stops and starts EC2 instances based on their tags using **Boto3** (AWS SDK for Python).

---

## **How It Works**
- The function fetches all EC2 instances with the **tag `Action`**.
- If an instance is tagged as **`Auto-Stop`**, it will be stopped.
- If an instance is tagged as **`Auto-Start`**, it will be started.
- It logs the **Instance IDs** of the affected EC2 instances.

---

## **Steps Followed**
### **1. Set Up Two EC2 Instances**
- Created **two EC2 instances** (`t2.micro`).
- Assigned the following **tags**:
  - **Instance 1:** `Key: Action`, `Value: Auto-Stop`
  - **Instance 2:** `Key: Action`, `Value: Auto-Start`

### **2. Created an IAM Role for Lambda**
- Created a role: **LambdaEC2ManagementRole**.
- Attached **AmazonEC2FullAccess** policy.

### **3. Created the Lambda Function**
- **Runtime:** Python 3.x
- **Permissions:** Assigned `LambdaEC2ManagementRole`.
- **Code:** Used **Boto3** to start/stop EC2 instances.

---

## **Lambda Function Code**
```python
import boto3

def lambda_handler(event, context):
    ec2 = boto3.client('ec2')

    # Fetch all EC2 instances with Auto-Stop or Auto-Start tags
    instances = ec2.describe_instances(Filters=[{'Name': 'tag:Action', 'Values': ['Auto-Stop', 'Auto-Start']}])

    stop_instances = []
    start_instances = []

    for reservation in instances['Reservations']:
        for instance in reservation['Instances']:
            instance_id = instance['InstanceId']
            state = instance['State']['Name']  # Get current instance state

            for tag in instance.get('Tags', []):
                if tag['Key'] == 'Action' and tag['Value'] == 'Auto-Stop' and state != 'stopped':
                    stop_instances.append(instance_id)
                elif tag['Key'] == 'Action' and tag['Value'] == 'Auto-Start' and state != 'running':
                    start_instances.append(instance_id)

    # Stop instances if they are running
    if stop_instances:
        ec2.stop_instances(InstanceIds=stop_instances)
        print(f"Stopping instances: {stop_instances}")

    # Start instances if they are stopped
    if start_instances:
        ec2.start_instances(InstanceIds=start_instances)
        print(f"Starting instances: {start_instances}")

    return {
        'statusCode': 200,
        'body': 'EC2 instance actions initiated successfully'
    }
