Running Labs for Demo Students
 


### Spring fileupload service: 

after we have updating the configuration of the yamlf ile 
we can reload the services by running this: 
```bash
docker compose up -d --force-recreate fileupload-srv
```
- You do not need `--build `because you're updating an external configuration file rather than Java source code or the Dockerfile.

- Validate the configurations 
```bash
docker compose exec fileupload-srv \
  cat /app/config/application.yaml
```


### Spring JPA Data 
```bash 
mkdir -p secrets

# Generate a random password
openssl rand -hex 32 > secrets/db_password.txt

# Restrict file permissions
chmod 600 secrets/db_password.txt
```


## Build Time Secrets - Dockerfile 
Used when building an image, for example, downloading dependencies from a private Maven repository.

Docker BuildKit supports secret mounts:
```Dockerfile
# syntax=docker/dockerfile:1

FROM eclipse-temurin:21-jdk AS build

WORKDIR /app

COPY . .

RUN --mount=type=secret,id=maven_token \
    cat /run/secrets/maven_token > /dev/null && \
    ./gradlew build --no-daemon
```
The example only demonstrates access to the secret. In a real project, Gradle would read it to authenticate with your private repository.

You can supply that build-time secret through Compose:
```yaml
services:
  restful-api:
    build:
      context: .
      secrets:
        - maven_token

secrets:
  maven_token:
    file: ./secrets/maven_token.txt
```
Build secrets are available only during the build step where they're mounted. They should not be copied into image layers or build outputs.

Never use Dockerfile `ARG` or `ENV` to store production database passwords.

### Additional best practices for your project
I would make these changes before deploying to production:
- Use a pinned PostgreSQL version rather than postgres:latest.
- Remove `container_name` unless you specifically need it; Compose already provides service DNS names such as `postgres`.
- Do not publish PostgreSQL port `5432` unless external database access is necessary.
- Run the Spring Boot application as a `non-root` user and verify volume permissions.
- Use a dedicated database user with only the permissions the application needs.
- Keep production secret files outside your Git repository and restrict host file access.
- Back up the PostgreSQL volume and uploaded files regularly.