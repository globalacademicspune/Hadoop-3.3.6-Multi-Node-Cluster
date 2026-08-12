# Hadoop 3.3.6 Multi-Node Cluster

This repository contains the setup and configuration of a **3-node Apache Hadoop 3.3.6 multi-node cluster** using **HDFS** for distributed storage and **YARN** for cluster resource management.

The cluster consists of **one NameNode and two DataNodes**. The complete installation and configuration procedure is documented step-by-step in `STEPS.md`.

---

## Cluster Architecture

```text
                    NameNode (nn)
                 ┌─────────────────┐
                 │    NameNode     │
                 │ ResourceManager │
                 └────────┬────────┘
                          │
                ┌─────────┴─────────┐
                │                   │
          ┌─────▼─────┐       ┌─────▼─────┐
          │    dn1    │       │    dn2    │
          │  DataNode  │       │  DataNode  │
          │ NodeManager│       │ NodeManager│
          └───────────┘       └───────────┘
```

---

## Cluster Details

| Node       | Hostname | Network    | Role                      |
| ---------- | -------- | ---------- | ------------------------- |
| NameNode   | `nn`     | Private IP | NameNode, ResourceManager |
| DataNode 1 | `dn1`    | Private IP | DataNode, NodeManager     |
| DataNode 2 | `dn2`    | Private IP | DataNode, NodeManager     |

> **Note:** This cluster uses **private IP addresses** for internal communication between nodes. Replace the private IP values in the setup with the private IP addresses assigned to your own nodes.

---

## Technologies Used

* **Apache Hadoop 3.3.6**
* **HDFS**
* **YARN**
* **Java 11**
* **Ubuntu Linux**
* **SSH**

---

## Hadoop Components

| Component           | Purpose                                          |
| ------------------- | ------------------------------------------------ |
| **NameNode**        | Manages HDFS metadata and filesystem namespace   |
| **DataNode**        | Stores the actual HDFS data blocks               |
| **ResourceManager** | Manages YARN cluster resources                   |
| **NodeManager**     | Manages resources and containers on worker nodes |
| **HDFS**            | Provides distributed and replicated storage      |
| **YARN**            | Provides cluster resource management             |

---

## Configuration Files

The following Hadoop configuration files are used in this setup:

| File            | Purpose                                                  |
| --------------- | -------------------------------------------------------- |
| `hadoop-env.sh` | Defines the Java environment used by Hadoop              |
| `core-site.xml` | Defines core Hadoop and default filesystem configuration |
| `hdfs-site.xml` | Defines HDFS replication and storage directories         |
| `yarn-site.xml` | Defines YARN configuration                               |
| `workers`       | Defines the worker/DataNode hosts                        |

---

## Setup Flow

```text
Configure Hostnames
        ↓
Configure /etc/hosts
        ↓
Configure Passwordless SSH
        ↓
Install Java 11
        ↓
Configure Environment Variables
        ↓
Install Hadoop 3.3.6
        ↓
Configure Hadoop
        ↓
Copy Hadoop to DataNodes
        ↓
Create HDFS Storage Directories
        ↓
Format NameNode
        ↓
Start HDFS
        ↓
Start YARN
        ↓
Verify Cluster
```

---

## Environment

| Component           | Version / Configuration |
| ------------------- | ----------------------- |
| Hadoop              | 3.3.6                   |
| Java                | 11                      |
| Cluster Size        | 3 Nodes                 |
| Storage             | HDFS                    |
| Resource Management | YARN                    |
| Operating System    | Ubuntu Linux            |

---

## Web UIs

After starting the Hadoop cluster, the following web interfaces can be used to monitor the cluster.

### NameNode Web UI

```text
http://nn:9870
```

The NameNode UI provides information about:

* HDFS filesystem
* DataNodes
* Storage
* Cluster status
* HDFS information

### YARN ResourceManager Web UI

```text
http://nn:8088
```

The ResourceManager UI provides information about:

* YARN cluster resources
* Worker nodes
* Running applications
* Resource usage
* Cluster activity

---

## Repository Structure

```text
hadoop-3.3.6-multi-node/
│
├── README.md
└── STEPS.md
```

### README.md

Provides an overview of the Hadoop cluster, architecture, components, configuration files, and setup flow.

### STEPS.md

Contains the complete **step-by-step installation and configuration procedure**, including the commands required to build the cluster and explanations of what each step does.

---

## Expected Cluster

After completing the setup, the expected service layout is:

```text
nn
├── NameNode
└── ResourceManager

dn1
├── DataNode
└── NodeManager

dn2
├── DataNode
└── NodeManager
```

The cluster is configured so that:

```text
HDFS  → Distributed Storage
YARN  → Resource Management
```

---

## Purpose

The purpose of this repository is to provide a practical reference for setting up and understanding a **multi-node Apache Hadoop 3.3.6 cluster**.

It demonstrates how multiple Linux nodes can be configured to work together as a Hadoop cluster using HDFS for distributed storage and YARN for resource management.

For the complete implementation, follow the instructions in **`STEPS.md`**.
