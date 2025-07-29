## Docker Networking Fundamentals

Docker networking allows you to connect containers to each other and to the outside world. Understanding how it works is crucial for building multi-container applications. Docker uses a pluggable system of network drivers to provide different types of networking.

### Container Networking Models

Docker's networking architecture is called the **Container Network Model (CNM)**. It provides a standard interface for network drivers. The main components are:

*   **Sandbox**: An isolated network stack for a container. It includes the container's Ethernet interfaces, ports, routing table, and DNS settings.
*   **Endpoint**: A virtual network interface that connects a sandbox to a network. A container can have multiple endpoints if it's connected to multiple networks.
*   **Network**: A software-defined group of endpoints that are allowed to communicate with each other.

### Docker Network Drivers

Network drivers are what provide the actual network implementation. Here are the most common ones:

#### 1. Bridge Network (`bridge`)

This is the **default** driver for standalone containers. It creates a private, internal network on the host machine.

*   **How it works**: Containers on the same bridge network can communicate using their internal IP addresses. To expose a container to the outside world, you must map a port from the container to a port on the Docker host.
*   **Key Feature**: Custom bridge networks provide automatic DNS resolution between containers, allowing them to communicate by name. The default `bridge` network does not have this feature.

**Example:** Run an Nginx web server and map host port 8080 to the container's port 80.

```bash
# Run the container with a port mapping
docker run -d --name my-web-server -p 8080:80 nginx

# Now, you can access the Nginx server in your browser at http://localhost:8080
# To clean up:
docker stop my-web-server && docker rm my-web-server
```

#### 2. Host Network (`host`)

This driver removes network isolation between the container and the Docker host.

*   **How it works**: The container shares the host's networking namespace. Any port a containerized application listens on is directly exposed on the host's IP address. No port mapping is needed.
*   **Use Case**: Useful when you need maximum network performance and port mapping is undesirable. However, it's a security risk as the container can access local host services and you can't run multiple containers on the same port.

**Example:** Run Nginx using the host network. It will use port 80 on your host.

```bash
# This command will fail if you already have a service running on port 80
docker run -d --name nginx-on-host --network host nginx

# Access the server directly at http://localhost:80
# To clean up:
docker stop nginx-on-host && docker rm nginx-on-host
```

#### 3. Overlay Network (`overlay`)

This driver is designed for multi-host networking and is the standard for **Docker Swarm**.

*   **How it works**: It creates a distributed network that spans across multiple Docker hosts, allowing containers (typically as part of a swarm service) on different hosts to communicate securely as if they were on the same private network.
*   **Use Case**: The backbone for communication between services in a Docker Swarm cluster.

**Example:** To create an overlay network, you first need to have a Docker Swarm initialized.

```bash
# 1. If you don't have a swarm, initialize one
docker swarm init

# 2. Create the overlay network
docker network create --driver overlay my-swarm-net
```

### Common Container Networking Commands

Here is a quick reference for managing Docker networks.

| Command                                        | Description                                                                                             |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `docker network ls`                            | Lists all available networks on the Docker host.                                                        |
| `docker network inspect <network_name>`        | Displays detailed information about a network, including connected containers.                          |
| `docker network create <network_name>`         | Creates a new custom bridge network by default. Use `--driver` to specify another type.                 |
| `docker network connect <net> <container>`     | Connects a running container to an additional network.                                                  |
| `docker network disconnect <net> <container>`  | Disconnects a container from a network.                                                                 |
| `docker network rm <network_name>`             | Removes a network. You cannot remove a network if containers are still connected to it.                 |

### Tutorial: Create and Use a Custom Network

The best practice for multi-container applications is to create a custom bridge network. This provides better isolation and, most importantly, allows containers to discover and communicate with each other using their names.

#### Step 1: Create a Custom Bridge Network

Let's create a network for our application.

```bash
docker network create my-app-network
```

#### Step 2: Run Containers and Attach Them to the Network

We'll run two `alpine` containers, simulating an "app" and a "db" service. We attach them to our network using the `--network` flag.

```bash
# Run a container named 'db'
docker run -dit --name db --network my-app-network alpine

# Run a container named 'app'
docker run -dit --name app --network my-app-network alpine
```
> **Note:** We use `alpine` with the `-dit` flags to run the containers in the background. By default, they will exit immediately. The `sleep` command could be added to keep them running for a longer test (e.g., `alpine sleep 3600`).

#### Step 3: Test Container-to-Container Communication

Now, from the `app` container, we can ping the `db` container *by its name*.

```bash
# Execute the ping command inside the 'app' container
docker exec app ping -c 3 db
```

You will see a successful reply, which proves that the `app` container was able to resolve the IP address of the `db` container using its name.

```
PING db (172.19.0.2): 56 data bytes
64 bytes from 172.19.0.2: seq=0 ttl=64 time=0.082 ms
64 bytes from 172.19.0.2: seq=1 ttl=64 time=0.123 ms
64 bytes from 172.19.0.2: seq=2 ttl=64 time=0.125 ms
```

#### Step 4: Clean Up

Once you're done, you can stop and remove the containers and the network.

```bash
# Stop and remove the containers
docker stop app db
docker rm app db

# Remove the network
docker network rm my-app-network
```