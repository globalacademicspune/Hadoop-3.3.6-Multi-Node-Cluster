# Step 1: Configure Hostnames

**Run on all nodes.**

Each node must have a unique hostname.

The hostname mapping used in this setup is:

```text
<PRIVATE_IP_NN>  nn
<PRIVATE_IP_DN1> dn1
<PRIVATE_IP_DN2> dn2
```

## 1.1 Set Hostname

On the NameNode:

```bash
sudo hostnamectl set-hostname nn
```

On DataNode 1:

```bash
sudo hostnamectl set-hostname dn1
```

On DataNode 2:

```bash
sudo hostnamectl set-hostname dn2
```

### What is happening?

The hostname identifies each machine inside the Hadoop cluster.

The expected mapping is:

```text
nn  → NameNode
dn1 → DataNode 1
dn2 → DataNode 2
```

Verify the hostname:

```bash
hostname
```

---

## 1.2 Configure `/etc/hosts`

Edit the hosts file on **all three nodes**:

```bash
sudo vi /etc/hosts
```

Add:

```text
<PRIVATE_IP_NN>  nn
<PRIVATE_IP_DN1> dn1
<PRIVATE_IP_DN2> dn2
```

### What is happening?

The `/etc/hosts` file provides local hostname resolution.

This allows the nodes to communicate using hostnames instead of directly using IP addresses.

For example:

```text
nn
dn1
dn2
```

can be resolved to their corresponding private IP addresses.

This is important because Hadoop configuration uses hostnames such as:

```text
hdfs://nn:9000
```

instead of hardcoding the NameNode IP address.

---

## 1.3 Verify Hostname Resolution

Run on all nodes:

```bash
getent hosts nn
```

```bash
getent hosts dn1
```

```bash
getent hosts dn2
```

Each hostname should resolve to the correct private IP address.

You can also test connectivity:

```bash
ping -c 2 nn
```

```bash
ping -c 2 dn1
```

```bash
ping -c 2 dn2
```

### Expected Result

All three nodes should be able to resolve the cluster hostnames correctly.

---

# Step 2: Configure Passwordless SSH

Hadoop uses SSH to communicate with worker nodes during cluster management and startup.

The required communication path is:

```text
             nn
            /  \
          SSH    SSH
          /        \
        dn1        dn2
```

## 2.1 Generate SSH Key

Run on the NameNode:

```bash
ssh-keygen
```

### What is happening?

This generates an SSH key pair:

```text
~/.ssh/id_rsa
~/.ssh/id_rsa.pub
```

The private key should remain on the NameNode.

The public key can be shared with the other nodes.

---

## 2.2 Configure Authorized Keys

The public key can be added to the authorized keys file:

```bash
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

### What is happening?

`authorized_keys` contains public keys that are allowed to authenticate using SSH keys.

For a multi-node Hadoop cluster, the public keys need to be exchanged/configured appropriately so that SSH access works between the cluster nodes.

---

## 2.3 Exchange Public Keys

Exchange the required public keys between the three nodes and ensure that SSH connectivity works.

Test from the NameNode:

```bash
ssh dn1
```

and:

```bash
ssh dn2
```

The NameNode should be able to connect to both DataNodes without requiring a password.

### Why is this required?

Hadoop startup scripts such as:

```bash
start-dfs.sh
```

and:

```bash
start-yarn.sh
```

communicate with worker nodes using SSH.

---

# Step 3: Java & Environment Setup

**Run on all nodes.**

Hadoop 3.3.6 requires Java to run its services.

## 3.1 Update Packages

```bash
sudo apt update
```

### What is happening?

This refreshes the local Ubuntu package index.

---

## 3.2 Upgrade Packages

```bash
sudo apt upgrade -y
```

### What is happening?

This upgrades installed packages to their available versions.

---

## 3.3 Install Java 11

```bash
sudo apt install -y openjdk-11-jdk
```

### What is happening?

Java 11 JDK is installed on each Hadoop node.

Verify Java:

```bash
java -version
```

All nodes should have Java 11 available.

---

## 3.4 Configure Hadoop Environment Variables

Edit `.bashrc`:

```bash
vi ~/.bashrc
```

Add:

```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export HADOOP_HOME=/opt/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
```

### What is happening?

Several environment variables are configured.

### `JAVA_HOME`

```text
/usr/lib/jvm/java-11-openjdk-amd64
```

Tells Hadoop where Java is installed.

### `HADOOP_HOME`

```text
/opt/hadoop
```

Defines the Hadoop installation directory.

### `PATH`

Adds Hadoop's executable directories to the shell's command path.

This allows commands such as:

```text
hdfs
hadoop
start-dfs.sh
start-yarn.sh
```

to be executed directly.

### `HADOOP_CONF_DIR`

```text
$HADOOP_HOME/etc/hadoop
```

Defines the location of Hadoop configuration files.

---

## 3.5 Reload Environment

```bash
source ~/.bashrc
```

### What is happening?

The updated `.bashrc` is loaded into the current shell session so the new environment variables become available.

---

# Step 4: Download & Configure Hadoop

**Perform the Hadoop installation and initial configuration on the NameNode (`nn`).**

## 4.1 Download Hadoop 3.3.6

```bash
wget https://downloads.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
```

### What is happening?

The Hadoop 3.3.6 compressed archive is downloaded from the Apache Hadoop distribution server.

The downloaded file is:

```text
hadoop-3.3.6.tar.gz
```

---

## 4.2 Extract Hadoop

```bash
tar -xvzf hadoop-3.3.6.tar.gz
```

### What is happening?

The compressed archive is extracted into a directory named:

```text
hadoop-3.3.6
```

---

## 4.3 Move Hadoop to `/opt`

```bash
sudo mv hadoop-3.3.6 /opt/hadoop
```

### What is happening?

The Hadoop installation is moved to:

```text
/opt/hadoop
```

This becomes the Hadoop installation directory used by the environment variables.

---

## 4.4 Change Ownership

```bash
sudo chown -R ubuntu:ubuntu /opt/hadoop
```

### What is happening?

Ownership of the Hadoop installation is assigned to the `ubuntu` user.

This allows the Hadoop installation and configuration to be managed by the `ubuntu` user.

---

## 4.5 Open Hadoop Configuration Directory

```bash
cd $HADOOP_HOME/etc/hadoop/
```

### What is happening?

This moves into Hadoop's main configuration directory.

The configuration files used in this setup are:

```text
hadoop-env.sh
core-site.xml
hdfs-site.xml
yarn-site.xml
workers
```

---

# Step 4.6: Configure `hadoop-env.sh`

Edit the file:

```bash
vi hadoop-env.sh
```

Set:

```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

### What is happening?

This explicitly tells Hadoop which Java installation should be used by Hadoop services.

---

# Step 4.7: Configure `core-site.xml`

Edit:

```bash
vi core-site.xml
```

Configure the default filesystem:

```xml
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://nn:9000</value>
</property>
```

### What is happening?

The `fs.defaultFS` property defines HDFS as the default filesystem.

```text
hdfs://nn:9000
```

means that the NameNode is available through:

```text
nn:9000
```

The hostname `nn` is resolved using the `/etc/hosts` configuration.

The basic flow is:

```text
Hadoop Client
      ↓
HDFS
      ↓
NameNode (nn:9000)
```

---

# Step 4.8: Configure `hdfs-site.xml`

Edit:

```bash
vi hdfs-site.xml
```

Configure:

```xml
<property>
    <name>dfs.replication</name>
    <value>2</value>
</property>

<property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/home/ubuntu/hdfs/namenode</value>
</property>

<property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/home/ubuntu/hdfs/datanode</value>
</property>
```

### What is happening?

This file defines HDFS-specific configuration.

### `dfs.replication`

```text
2
```

Defines the number of replicas HDFS should maintain for each data block.

With two DataNodes, the basic storage layout is:

```text
HDFS Block
    │
    ├──→ dn1
    │
    └──→ dn2
```

### `dfs.namenode.name.dir`

```text
/home/ubuntu/hdfs/namenode
```

Defines where the NameNode stores its filesystem metadata.

### `dfs.datanode.data.dir`

```text
/home/ubuntu/hdfs/datanode
```

Defines where DataNodes store HDFS data blocks.

---

# Step 4.9: Configure `yarn-site.xml`

Edit:

```bash
vi yarn-site.xml
```

Configure:

```xml
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
```

### What is happening?

This enables the MapReduce shuffle service on the NodeManager.

The shuffle phase is part of MapReduce processing where intermediate data is transferred from mapper tasks toward reducer tasks.

Simplified flow:

```text
Mapper
  ↓
Intermediate Data
  ↓
Shuffle
  ↓
Reducer
```

---

# Step 4.10: Configure `workers`

Edit:

```bash
vi workers
```

Add:

```text
dn1
dn2
```

### What is happening?

The `workers` file identifies the worker nodes in the Hadoop cluster.

In this setup:

```text
nn
│
├── dn1
└── dn2
```

Hadoop startup scripts use the worker information when starting services across the cluster.

---

# Step 5: Copy Hadoop from NameNode to DataNodes

The Hadoop installation and configuration need to be available on both DataNodes.

## 5.1 Prepare Hadoop Directory on `dn1`

From the NameNode:

```bash
ssh dn1 sudo mkdir -p /opt/hadoop && sudo chown ubuntu:ubuntu /opt/hadoop
```

### What is happening?

This creates the Hadoop installation directory on `dn1` and assigns ownership to the `ubuntu` user.

---

## 5.2 Prepare Hadoop Directory on `dn2`

```bash
ssh dn2 sudo mkdir -p /opt/hadoop && sudo chown ubuntu:ubuntu /opt/hadoop
```

The same Hadoop directory structure is prepared on `dn2`.

---

## 5.3 Copy Hadoop to `dn1`

From the NameNode:

```bash
scp -r /opt/hadoop/* ubuntu@dn1:/opt/hadoop/
```

### What is happening?

This copies the Hadoop installation and configuration files from the NameNode to DataNode 1.

---

## 5.4 Copy Hadoop to `dn2`

```bash
scp -r /opt/hadoop/* ubuntu@dn2:/opt/hadoop/
```

### What is happening?

This copies the same Hadoop installation and configuration to DataNode 2.

At the end of this step:

```text
nn  → /opt/hadoop
dn1 → /opt/hadoop
dn2 → /opt/hadoop
```

All three nodes have the Hadoop installation.

---

# Step 6: Create HDFS Storage Directories

HDFS requires directories for storing NameNode metadata and DataNode blocks.

## 6.1 Create Directories on NameNode

Run on `nn`:

```bash
mkdir -p ~/hdfs/namenode ~/hdfs/datanode
```

### What is happening?

Two directories are created:

```text
~/hdfs/namenode
~/hdfs/datanode
```

The NameNode uses the `namenode` directory for its metadata, while the `datanode` directory is available for DataNode storage.

---

## 6.2 Create Directories on `dn1`

From the NameNode:

```bash
ssh dn1 mkdir -p ~/hdfs/namenode ~/hdfs/datanode
```

### What is happening?

The required HDFS storage directories are created on DataNode 1.

---

## 6.3 Create Directories on `dn2`

```bash
ssh dn2 mkdir -p ~/hdfs/namenode ~/hdfs/datanode
```

### What is happening?

The required HDFS storage directories are created on DataNode 2.

The resulting directory structure is:

```text
nn
└── ~/hdfs/
    ├── namenode/
    └── datanode/

dn1
└── ~/hdfs/
    ├── namenode/
    └── datanode/

dn2
└── ~/hdfs/
    ├── namenode/
    └── datanode/
```

---

# Step 7: Format and Start Hadoop

**Run these commands only on the NameNode (`nn`).**

## 7.1 Format the NameNode

```bash
hdfs namenode -format
```

### What is happening?

This initializes the NameNode filesystem metadata for the new HDFS cluster.

The NameNode must be formatted before HDFS can be started for the first time.

> **Important:** Do not run this command on an existing HDFS cluster unless you intentionally want to reinitialize the NameNode. Formatting can make existing HDFS metadata inaccessible.

---

## 7.2 Start HDFS

```bash
start-dfs.sh
```

### What is happening?

This starts the HDFS services across the cluster.

The expected HDFS architecture is:

```text
                 NameNode
                    │
           ┌────────┴────────┐
           ▼                 ▼
        DataNode          DataNode
          dn1                dn2
```

The NameNode manages the HDFS namespace and metadata, while the DataNodes store the actual HDFS blocks.

---

## 7.3 Start YARN

```bash
start-yarn.sh
```

### What is happening?

This starts the YARN resource management services.

The expected YARN architecture is:

```text
             ResourceManager
                    │
           ┌────────┴────────┐
           ▼                 ▼
       NodeManager       NodeManager
           dn1                dn2
```

The ResourceManager manages cluster resources while NodeManagers manage resources and containers on the worker nodes.

---

# Step 8: Verify the Hadoop Cluster

After starting HDFS and YARN, verify the running Hadoop processes.

Run on each node:

```bash
jps
```

---

## Expected Output on `nn`

The NameNode should have:

```text
NameNode
ResourceManager
```

Depending on the Hadoop service configuration, additional Hadoop processes may also be visible.

---

## Expected Output on `dn1`

Expected:

```text
DataNode
NodeManager
```

---

## Expected Output on `dn2`

Expected:

```text
DataNode
NodeManager
```

---

# Step 9: Verify the Web UIs

Hadoop 3.x uses the following default web UI ports.

## NameNode Web UI

```text
http://nn:9870
```

### Port

```text
9870
```

The NameNode UI provides information about:

* HDFS status
* DataNodes
* Storage
* Filesystem information
* Cluster health

---

## YARN ResourceManager Web UI

```text
http://nn:8088
```

### Port

```text
8088
```

The ResourceManager UI provides information about:

* YARN cluster resources
* NodeManagers
* Running applications
* Resource usage
* Cluster activity

---

# Final Cluster Layout

After completing all the steps, the cluster should have the following architecture:

```text
                         nn
                   ┌───────────────┐
                   │   NameNode    │
                   │ ResourceMgr   │
                   └───────┬───────┘
                           │
               ┌───────────┴───────────┐
               │                       │
               ▼                       ▼
              dn1                     dn2
         ┌────────────┐          ┌────────────┐
         │  DataNode  │          │  DataNode  │
         │ NodeManager│          │ NodeManager│
         └────────────┘          └────────────┘
```

## Service Summary

| Node  | Services                  |
| ----- | ------------------------- |
| `nn`  | NameNode, ResourceManager |
| `dn1` | DataNode, NodeManager     |
| `dn2` | DataNode, NodeManager     |

---

# Setup Complete

At this point, the Hadoop 3.3.6 multi-node cluster has been configured with:

```text
HDFS
  ↓
Distributed Storage

YARN
  ↓
Cluster Resource Management
```

The cluster is now ready for HDFS operations and YARN-based workloads.

For the project overview and architecture, refer to `README.md`.
