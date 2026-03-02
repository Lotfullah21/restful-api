# Setting Up a Redis Instance in GCP Memorystore

### 1. Navigate to Memorystore

- Go to the [GCP Console](https://console.cloud.google.com/).
- Navigate to the **Memorystore** page.

---

### 2. Create a Redis Instance

1. **Click "Create Instance"**.
2. **Instance ID**: Provide a unique name for your Redis instance (e.g., `my-redis`).
3. **Tier**:
   - Select either **Standard** (for high availability) or **Basic** (single node).
4. **Region and Zone**:
   - Choose a region close to your application's deployment.
5. **Network**:
   - Select the same **VPC network** as your Django application.

---

### 3. Configure Other Options

- **Redis Version**: Choose the latest supported version.
- **Size**: Select a size based on your expected cache usage.

---

### 4. Create the Instance

- Click **Create** and wait for the Redis instance to be provisioned.

---

### 5. Note the Redis Host

- Once the instance is created, copy the **host IP address** (e.g., `10.0.0.3`).
