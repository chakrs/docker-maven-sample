# docker-maven-sample

This is a simple spring boot application that is dockerized with maven docker build plugin.

You will need all of the following components to successfully build and run this application locally following the the steps in this README:

| [Java 8](http://java.oracle.com/) | [Maven 3](http://maven.apache.org/) | [Git](https://github.com/) | [Docker](https://www.docker.com/) |

In addition, to push the image to a private registry either have your own registry running somewhere or create a registry in Azure. For that you will need an Azure subscription; if you don't already have an Azure subscription, you can sign up for a [free Azure account](https://azure.microsoft.com/pricing/free-trial/) and deploy your Azure Container Registry. The registry references in the `pom.xml` in this project uses an Azure Container Registry.


It uses the following `Dockerfile`
```
FROM frolvlad/alpine-oraclejdk8:slim
VOLUME /tmp
ARG JAR_FILE_NAME
COPY $JAR_FILE_NAME app.jar
RUN sh -c 'touch /app.jar'
ENV JAVA_OPTS=""
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar" ]
```
## Build and Push Image
Since the Docker build process integrated with the Maven build process and it binds the default phases, we can type `mvn package` and get a Docker image. When use `mvn deploy`, the image gets pushed to the registry. Also we can just say `mvn dockerfile:build dockerfile:push` to build and push the image to the registry.

## This sample demonstrates the following two features

### Authentication with private Docker registry

#### Authenticating with maven settings.xml

Since version `1.3.0` (we use the latest `1.3.6` version in this project), private registry authentication can be done using maven `settings.xml` instead of docker configuration in `~/.dockercfg` or `~/.docker/config.json` file when pulling, pushing, or building images to private registries. This is configured in the `pom.xml` with the following:
```
<configuration>
  <serverId>${docker.image.prefix}</serverId>
  <registryUrl>https://${docker.image.prefix}</registryUrl>
  <useMavenSettingsForAuth>true</useMavenSettingsForAuth>
  <repository>${docker.image.prefix}/${project.artifactId}</repository>
  <tag>${project.version}</tag>
</configuration>
```

Here we provide the `serverId` and `registryUrl` also set `useMavenSettingsForAuth` tag value to `true`. This allows the maven docker plugin to read authentication information like `usename` and `password` from the maven `settings.xml` file.

Then, in the maven settings file, add configuration for the server:

```
<servers>
  <server>
    <id>registryName.example.com</id>
      <username>myUserName</username>
      <password>myPassword</password>
  </server>
</servers>
```

### Dynamically provide arguments to the `Dockerfile`

Since in most cases our jar file name would change form build to build i.e. adding different version number to the file name, we can not use a static name in the `Dockerfile` for the jar file name. The following `pom.xml` `maven` `configuration` setting helps us with providing build arguments for the `Dockerfile`

```
<configuration>
  <buildArgs>
    <JAR_FILE_NAME>${project.build.finalName}.jar</JAR_FILE_NAME>
  </buildArgs>
</configuration>
```

The `JAR_FILE_NAME` is referenced in the `Dockerfile` where the value of `JAR_FILE_NAME` is  `${project.build.finalName}.jar`.

The `Dockerfile` argument passing feature comes with the docker client version `8.8.4` that's why we had to override the docker client version in the `dockerfile-maven-plugin` setup within the `plugin` section.
