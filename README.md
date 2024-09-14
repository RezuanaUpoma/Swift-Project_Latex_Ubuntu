# Swift-Project_Latex_Ubuntu
Setting up OpenStack Swift and running i/o operations for our thesis findings


# OpenStack Swift Setup with Erasure Coding

This guide outlines the installation and configuration of OpenStack Swift with erasure coding, including commands for operating on Swift.

## Table of Contents
1. [Overview](#Overview)
2. [Installation](#installation)
3. [Erasure Coding Setup](#erasure-coding-setup)
4. [Operating Swift](#operating-swift)
5. [Commands Reference](#commands-reference)

## Description 

In our thesis work, we aimed to evaluate the performance and fault tolerance of erasure coding in OpenStack Swift. To achieve this, we conducted a series of experiments to assess the system's ability to handle data under various conditions. Our objectives included uploading, downloading, and deleting data from Swift using an erasure coding policy to measure its efficiency and reliability.

**Project Description:**

To proceed with our thesis, it was essential to analyze the performance of erasure coding in an object storage system. We set up OpenStack Swift on both a remote server and a local server, ensuring that all dependencies were correctly installed and configured. We implemented an erasure coding policy to understand its impact on data management operations.

The experimental setup involved:

1. **Environment Setup:** We installed and configured OpenStack Swift on Ubuntu, including all required dependencies for erasure coding.
   
2. **Erasure Coding Configuration:** We defined and applied an erasure coding policy in Swift, setting parameters such as the number of data and parity fragments and the object segment size.

3. **Data Upload and Management:** We performed upload, download, and delete operations on two datasets: a self-made dataset and the COCO-17 dataset. These operations were executed to evaluate the system’s performance and fault tolerance under erasure coding.

4. **Performance Evaluation:** We measured the I/O performance by recording the timing for each upload, download, and delete operation for every file. This allowed us to analyze the efficiency and reliability of the erasure coding policy.

5. **Data and Results:** The repository includes all the relevant databases and configuration files used during the experiments, providing a comprehensive overview of the setup and results.

Our findings contribute to understanding how erasure coding impacts object storage systems and provide insights into its effectiveness in real-world scenarios.

## Installation

### 1. Install Swift

1. **Install Swift and Required Packages:**

    ```bash
    sudo apt-get update
    sudo apt-get install -y swift-proxy-server swift-object-server swift-container-server swift-account-server
    ```

2. **Install Swift Dependencies:**

    ```bash
    sudo apt-get install -y python-swiftclient python-keystoneclient
    ```

3. **Configure Swift Services:**

    - Edit configuration files in `/etc/swift/`:
      - `proxy-server.conf`
      - `object-server.conf`
      - `container-server.conf`
      - `account-server.conf`

4. **Initialize Swift Rings:**

    ```bash
    swift-ring-builder object-3.builder create 10 14 1
    swift-ring-builder container-3.builder create 10 14 1
    swift-ring-builder account-3.builder create 10 14 1
    ```

5. **Add Devices to Rings:**

    ```bash
    swift-ring-builder object-3.builder add r1z1-127.0.0.1:6000/d1 100
    swift-ring-builder container-3.builder add r1z1-127.0.0.1:6000/d1 100
    swift-ring-builder account-3.builder add r1z1-127.0.0.1:6000/d1 100
    ```

6. **Rebalance Rings:**

    ```bash
    swift-ring-builder object-3.builder rebalance
    swift-ring-builder container-3.builder rebalance
    swift-ring-builder account-3.builder rebalance
    ```

## Erasure Coding Setup

### 1. Configure Erasure Coding Policy

1. **Edit Erasure Coding Policy:**

    Add the following to the Swift configuration file (e.g., `/etc/swift/swift.conf`):

    ```ini
    [storage-policy:3]
    name = deepfreeze10-4
    policy_type = erasure_coding
    ec_type = liberasurecode_rs_vand
    ec_num_data_fragments = 10
    ec_num_parity_fragments = 4
    ec_object_segment_size = 1048576
    ```

2. **Create and Rebalance Erasure Coding Ring:**

    ```bash
    swift-ring-builder object-3.builder rebalance
    ```

## Operating Swift

### Authentication

1. **Authenticate with Swift:**

    ```bash
    curl -v -H 'X-Auth-User: admin:admin' -H 'X-Auth-Key: admin' http://localhost:8080/auth/v1.0/
    ```

### Upload and Manage Files

1. **Upload a File:**

    ```bash
    curl -i -X PUT -H 'X-Auth-Token: <auth_token>' "http://127.0.0.1:8080/v1.0/AUTH_admin/mycontainer/Stext1.txt" -T /home/upom/Desktop/file.txt
    ```

2. **Retrieve a File:**

    ```bash
    curl -v -X GET -H 'X-Storage-Token: <auth_token>' "http://127.0.0.1:8080/v1.0/AUTH_admin/mycontainer/Stext1.txt"
    ```

3. **Download a File to Local System:**

    ```bash
    curl -v -X GET -H 'X-Storage-Token: <auth_token>' "http://127.0.0.1:8080/v1.0/AUTH_admin/mycontainer/Stext1.txt" -o /home/upom/Desktop/T.txt
    ```

4. **Delete a File:**

    ```bash
    curl -v -X DELETE -H 'X-Auth-Token: <auth_token>' http://127.0.0.1:8080/v1.0/AUTH_admin/mycontainer/Stext1.txt
    ```

### File Distribution Check

1. **Check File Distribution:**

    ```bash
    swift-get-nodes /etc/swift/object-2.ring.gz AUTH_admin ec-container text1.txt
    swift-get-nodes /etc/swift/object-2.ring.gz AUTH_admin ec-container mp31.mp3
    ```

2. **Check Particular Fragment:**

    ```bash
    cat /srv/node/d9/objects-2/748/251//bb1bec31890a703f1cff86605e5a0251/1677059693.44381#13#d.data
    ```

### Container Operations

1. **Create an EC Container:**

    ```bash
    curl -v -X PUT -H 'X-Auth-Token: <auth_token>' http://127.0.0.1:8080/v1.0/AUTH_admin/ec-container -H 'X-Storage-Policy: deepfreeze10-4'
    ```

2. **List Container Contents:**

    ```bash
    curl -v -X GET -H 'X-Auth-Token: <auth_token>' http://127.0.0.1:8080/v1.0/AUTH_admin/ec-container
    ```

3. **Put File Inside EC Container:**

    ```bash
    curl -v -X PUT -H 'X-Auth-Token: <auth_token>' http://127.0.0.1:8080/v1.0/AUTH_admin/ec-container/1.jpg -T /path/to/file.jpg
    ```

4. **Replicate Object:**

    ```bash
    swift-object-replicator --once --object ec-container 1.jpg
    ```

## Commands Reference

- **List All Containers**: 

    ```bash
    curl -v -X GET -H 'X-Auth-Token: <auth_token>' http://127.0.0.1:8080/v1.0/AUTH_admin
    ```

- **Get Container Info**: 

    ```bash
    curl -v -X GET -H 'X-Auth-Token: <auth_token>' http://127.0.0.1:8080/v1.0/AUTH_admin/<container_name>
    ```

---

Replace `<auth_token>` with your actual authentication token and adjust paths as needed.
