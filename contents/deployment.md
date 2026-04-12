---
title: "Deployment"
description: "Helpful tips and best practices for deploying your Supranim applications to production environments, ensuring optimal performance and reliability."
---

## Server Configuration
When deploying your Supranim application, it's important to configure your server properly to ensure optimal performance and security. Here are some tips for configuring your server:

- Use a reverse proxy like Nginx or Apache to handle incoming requests and forward them to your Supranim application. This is recommended if you plan to serve multiple websites from the same server or if you want to take advantage of features like load balancing and SSL termination provided by the reverse proxy.

- Configure SSL/TLS to secure your application and protect user data. You can use services like Let's Encrypt to obtain free SSL certificates for your domain.

## Release Mode
When deploying your Supranim application, it's recommended to build it in **release mode** for optimal performance:

```bash
nimble build -d:release
```

This will compile your application with optimizations enabled, resulting in a faster and more efficient executable for production environments.


## Deploying with Supra CLI
We are working on adding deployment capabilities to the Supra CLI, which will allow you to easily deploy your Supranim applications to various hosting providers and platforms. Stay tuned for updates on this feature!