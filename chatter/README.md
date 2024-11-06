# ROS 2 Chatter Example with Docker Compose

This repository contains a Docker Compose setup for running a ROS 2 Humble chatter example with three containers: a Discovery Server, talker, and listener. The setup demonstrates both multicast-based and Discovery Server-based communication methods.

## Prerequisites

- Docker
- Docker Compose

## Files

- `Dockerfile`: Builds a Docker image with the `demo_nodes_cpp` package installed.
- `docker-compose.host-network.yaml`: Docker Compose file using host network mode (multicast-based discovery)
- `docker-compose.discovery.yaml`: Docker Compose file using Discovery Server (unicast-based discovery)
- `fastdds_discovery_server.xml`: Configuration file for Fast-DDS Discovery Server settings

## Discovery Server Setup

The repository provides two different approaches for ROS 2 node discovery:

### 1. Host Network Mode (Simple Discovery Protocol)
Uses multicast for node discovery. All containers share the host's network stack, enabling automatic discovery through multicast packets.

### 2. Discovery Server Mode
Uses a centralized Discovery Server for node discovery, eliminating the need for multicast. This is particularly useful when:
- Multicast is not available or reliable
- Network isolation is required
- Working across different networks or cloud environments

#### Key Components:

1. **Discovery Server Container**:
   - Acts as a central registry for ROS 2 nodes
   - Runs on port 11811 (UDP)
   - Generates a unique GUID prefix (visible in logs)
   - Example: `fastdds discovery --server-id 0`

2. **Client Nodes (Talker/Listener)**:
   - Connect to Discovery Server using its address and port
   - Can be configured as either:
     - `CLIENT`: Only communicates through the server
     - `SUPER_CLIENT`: Can establish direct connections after discovery

3. **Network Configuration**:
   - Each service can run on isolated networks
   - All services need access to Discovery Server port (11811)
   - DDS communication ports (7400-7402) for data exchange

#### Fast-DDS XML Configuration:
```xml
<discoveryProtocol>CLIENT</discoveryProtocol>
<!-- or -->
<discoveryProtocol>SUPER_CLIENT</discoveryProtocol>

<!-- Server identification -->
<RemoteServer prefix="44.53.00.5f.45.50.52.4f.53.49.4d.41">
    <!-- Default eProsima Discovery Server GUID -->
```

## Usage

1. **Using Host Network (Multicast)**:
    ```bash
    docker-compose -f docker-compose.host-network.yaml up --build
    ```

2. **Using Discovery Server (Unicast)**:
    ```bash
    docker-compose -f docker-compose.discovery.yaml up --build
    ```

## Discovery Server vs Host Network

### Discovery Server Advantages:
- Works without multicast support
- Better network isolation
- More control over discovery process
- Suitable for cloud deployments
- Can work across different networks

### Host Network Advantages:
- Simpler setup
- Lower latency
- No single point of failure
- Automatic peer discovery
- Better for local development

## Troubleshooting

1. **Verify Discovery Server:**
   ```bash
   docker logs ros2_discovery_server
   # Look for GUID prefix and server status
   ```

2. **Check Client Connectivity:**
   ```bash
   docker exec -it ros2_talker ros2 node list
   docker exec -it ros2_listener ros2 node list
   ```

3. **Common Issues:**
   - Incorrect GUID prefix in XML configuration
   - Port 11811 not accessible
   - Network isolation preventing discovery
   - XML configuration parsing errors

## References

- [ROS 2 Discovery Server Documentation](https://docs.ros.org/en/humble/Tutorials/Advanced/Discovery-Server/Discovery-Server.html)
- [Fast-DDS Discovery Server](https://fast-dds.docs.eprosima.com/en/latest/fastdds/discovery/discovery_server.html)
- [ROS 2 Network Configuration](https://docs.ros.org/en/humble/Concepts/About-Different-Middleware-Vendors.html)
