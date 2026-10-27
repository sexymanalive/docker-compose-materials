Running Labs for Demo Students
 


- Spring fileupload service: 
after we have updating the configuration of the yamlf ile 
we can reload the services by running this: 
```bash
docker compose up -d --force-recreate fileupload-srv

```
- You do not need `--build `because you're updating an external configuration file rather than Java source code or the Dockerfile.