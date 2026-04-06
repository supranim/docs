---
title: "Dependencies"
description: "Discover the essential dependencies required for Supranim to function properly and ensure a smooth development experience."
---

### Nim packages and libraries
Supranim relies on several Nim packages and libraries to provide its powerful features and functionality. Most of these dependencies are packages that I maintain and have been developed specifically for Supranim, while others are popular third-party libraries such as Libevent and Monocypher.

All in-house packages are available on GitHub, on [Supranim's GitHub org page](https://github.com/supranim), or on [OpenPeeps GitHub org page](https://github.com/openpeeps) for those pretty framework-agnostic libraries.

For a complete list of package dependencies, please refer to the [Nimble file](https://github.com/supranim/supranim) in the Supranim repository. Also, depending on the project type you choose when creating a new project, some additional dependencies may be required. 🔥 [Check the Starter Kits documentation](/starter-kits) for more details on the specific dependencies for each project type.

### C Library Dependencies 
Supranim is a web framework built on top of [Libevent notification library](https://github.com/libevent/libevent), a high-performance, well-known C library that provides asynchronous event notification. This allows Supranim to handle a large number of concurrent connections efficiently, making it ideal for building high-performance web applications.

### Building Libevent from Source
To ensure you have the latest version of Libevent, you can build it from source. Follow these steps:
```bash
# Clone the LibEvent repository
git clone https://github.com/libevent/libevent

# Navigate to the libevent directory
cd libevent && mkdir build && cd build
cmake .. && make

# Install the library globally
sudo make install
```
