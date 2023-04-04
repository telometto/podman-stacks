# Podman Stacks

A personal collection of useful Podman stacks and containers for various applications and services. This repository includes configurations for media management, database management, and more.

## Table of Contents

- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Usage](#usage)
- [Pods and Containers](#pods-and-containers)
- [SELinux Labels](#selinux-labels)
- [Rootless Containers](#rootless-containers)
- [Contributing](#contributing)
- [Issues and Questions](#issues-and-questions)
- [License](#license)


## Getting Started

### Prerequisites

- Podman installed on your system. You can follow the [official installation guide](https://podman.io/getting-started/installation) to set up Podman.
- Basic understanding of YAML, Podman, and containerized applications.

#### Required packages
- Fedora:
  ```bash
  dnf install podman podman-compose podman-docker
  ```

### Usage

1. Clone this repository:

```bash
git clone https://github.com/telometto/podman-stacks.git
cd podman-stacks
```

2. Create a `.env` file in the root directory of the repository. This file should include any secret variables required by the pods and containers. For example:

```
MYSQL_ROOT_PASSWORD="your_root_password"
MYSQL_USER="your_mysql_user"
MYSQL_PASSWORD="your_mysql_user_password"
```
**Note:** If you have any special chars inside any of the variables, be sure to wrap them in double-quotes (`"`)!

3. Choose the stack you want to deploy and run the appropriate Podman command:

```bash
podman play kube /path/to/stack.yml
```

## Pods and Containers
This repository currently includes the following stacks:
- Databases:
  - `mysql_stack.yml`: MySQL database, MySQL Workbench, and phpMyAdmin.
- Music:
  - `music_tagging_stack.yml`: Music tagging and management that uses [Beets](https://github.com/beetbox/beets), [Lidarr](https://github.com/Lidarr/Lidarr), [NZBHydra 2](https://github.com/theotherp/nzbhydra2), and [Jackett](https://github.com/Jackett/Jackett).

## SELinux Labels

When mounting volumes in your Podman containers on SELinux-enabled systems, it is important to consider SELinux labels:

- `:Z` - Use this label when a volume is shared between multiple containers within the same pod, and these containers require access to the same files. This label ensures proper SELinux context and isolation.
- `:z` - Use this label when a volume is shared between multiple containers across different pods, and these containers require access to the same files. This label provides a more permissive SELinux context, allowing access from multiple pods.
- No label - Use this when the volume is used by a single container and not shared with others.

**Note:** If you are using a non-SELinux-enabled distribution, you can ignore these labels, as they are not required for managing access control on such systems.

## Rootless Containers

The containers in this repository are configured to run as rootless containers by default, meaning they do not require escalated privileges. Running rootless containers provides an additional layer of security by minimizing the potential attack surface.

However, if you wish to run the containers with elevated privileges, you can do so by using the `--privileged` flag when running the Podman command:

```bash
podman play kube --privileged /path/to/stack.yml
```

**Note:** Ports below 1024 are considered privileged ports and *require* elevated privileges to bind to. To run containers that expose privileged ports, you may need to run the containers with escalated privileges or configure the container to use a higher port number.

## Contributing
Contributions are welcome! Feel free to submit pull requests or open issues to suggest new stacks, improvements, or fixes.

## Issues and Questions

If you encounter any issues or have questions regarding the usage of these Podman stacks and containers, please feel free to open an issue in the repository. When submitting an issue, provide as much information as possible to help us understand and resolve your problem. Include details about your system, the specific stack or container you're using, and any error messages or logs.

Before submitting an issue, please make sure to check the existing issues to avoid duplicate submissions. You can also search the closed issues to see if your question has been addressed previously.

## License
This project is licensed under the GNU Affero GPLv3.