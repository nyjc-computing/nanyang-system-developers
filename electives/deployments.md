# Deployments

[Containers](./containers.md) are isolated environments that package an application and its dependencies together, allowing it to run consistently across different systems. They are widely used in modern software development and deployment, especially in cloud environments.

# Containers in the Cloud

Many cloud platforms use containers to host applications. For example:

- **Google Cloud Engine** and **Replit** both use containers to run your code in isolated environments. This means your app gets its own environment, and you don't have to worry about conflicting with other users' code.
- **GitHub Codespaces** provides a development environment in the cloud using containers, allowing you to work on your code without needing to set up anything locally.
- **GitHub Actions** uses containers to run workflows, ensuring that your automated workflows run in a consistent environment every time.

# Deploying an Application

When you deploy an application, you want to ensure it runs the same way in production as it does in development.

The simplest way to do this is to run the application on a computer. However, when multiple applications need to run on the same computer, they can interfere with each other. For example, if one application requires a specific version of Python or a library, it might conflict with another application that needs a different version.

We use **containers** to enable multiple applications with conflicting requirements to run on the same computer.

# Terms

A **DockerFile** is a text file that contains instructions for building a Docker container image. It specifies the base image, the software to install, and any configuration needed for the application.

A **Docker image** is a snapshot of a container that includes the application code, runtime, libraries, and dependencies. It is built from a DockerFile and can be shared or deployed.

A **Docker container** is a running instance of a Docker image. It is an isolated environment where the application runs, with its own filesystem, processes, and network stack.

# Deployment with Containers

When we deploy an application using containers, e.g. when deploying on Replit, the following steps typically occur:

1. **Build the Docker Image**: The DockerFile is used to create a Docker image that contains the application and its dependencies.
2. **Push the Image to a Registry**: The Docker image is pushed to a container registry (like Docker Hub or Google Container Registry) where it can be accessed by the deployment platform.
3. **Deploy the Container**: The deployment platform pulls the Docker image from the registry and runs it as a container. This ensures that the application runs in a consistent environment, regardless of where it is deployed.
4. **Run the Application**: The container runs the application, and it can be accessed via a URL or an API endpoint.

# Further Reading

- [Docker Documentation](https://docs.docker.com/get-started/)
- [Google Cloud Container Registry](https://cloud.google.com/container-registry)
- [Replit Deployments](https://docs.replit.com/category/replit-deployments)
