# ROS 2 Chatter Example with Docker Compose

This repository contains a Docker Compose setup for running a ROS 2 Humble chatter example with two containers: one for the talker and one for the listener. The setup ensures proper communication between the ROS 2 nodes using the host network, IPC, and PID namespaces.

## Prerequisites

- Docker
- Docker Compose

## Files

- `Dockerfile`: Builds a Docker image with the `demo_nodes_cpp` package installed.
- `docker-compose.host-network.yaml`: Docker Compose file to run the ROS 2 talker and listener with the necessary configurations.

## Usage

1. **Build and run the containers**:
    ```bash
    docker-compose -f docker-compose.host-network.yaml up --build
    ```

2. **Verify the ROS 2 nodes and topics**:

    - **In the talker container**:
        ```bash
        docker exec -it ros2_talker /bin/bash
        source /opt/ros/humble/setup.bash
        ros2 node list
        ros2 topic list
        ```

    - **In the listener container**:
        ```bash
        docker exec -it ros2_listener /bin/bash
        source /opt/ros/humble/setup.bash
        ros2 node list
        ros2 topic list
        ```

3. **Check communication between the nodes**:

    - **Publish a message from the talker**:
        ```bash
        docker exec -it ros2_talker /bin/bash
        source /opt/ros/humble/setup.bash
        ros2 topic pub /chatter std_msgs/String "data: 'Hello, world!'"
        ```

    - **Echo the message in the listener**:
        ```bash
        docker exec -it ros2_listener /bin/bash
        source /opt/ros/humble/setup.bash
        ros2 topic echo /chatter
        ```

## Explanation

### Network Mode: Host

- **Shared Network Namespace**: Using `network_mode: host` allows the container to share the host's network namespace. This means the container can use the host's IP address and network interfaces directly, which is essential for ROS 2 nodes to communicate over the network as if they were running on the host machine.

### IPC Mode: Host

- **Shared IPC Namespace**: The `ipc: host` option enables the container to share the host's IPC namespace. ROS 2 and its underlying DDS implementation use IPC mechanisms like shared memory and semaphores for efficient inter-process communication. Sharing the IPC namespace ensures nodes in different containers can communicate using these mechanisms.

### PID Mode: Host

- **Shared PID Namespace**: The `pid: host` option allows the container to share the host's PID namespace. This means processes inside the container can see and interact with processes on the host. Some ROS 2 features rely on PID-based identification and management of processes, making this option necessary.

### Why These Options Matter for ROS 2

- **Node and Service Discovery**: ROS 2 relies on the Data Distribution Service (DDS) for node and service discovery. DDS implementations (like Fast DDS) often use shared memory and other IPC mechanisms to optimize communication. When running in isolated namespaces, the discovery process can fail because the nodes cannot see each other’s IPC resources.
- **Process Coordination**: Some functionalities in ROS 2 might rely on being able to see or signal other processes, which requires a shared PID namespace.

### Summary

By setting `network_mode: host`, `ipc: host`, and `pid: host`, you ensure that ROS 2 nodes in different containers can effectively discover and communicate with each other using shared network, IPC, and PID namespaces. This setup mimics running all nodes on the same physical machine, removing the isolation that Docker typically imposes.

## References

- https://github.com/rosblox/ros-template
