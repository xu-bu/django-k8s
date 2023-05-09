# local deploying testing
1. Create python venv and install pkgs (open a cmd in root folder):
    ```
    python -m venv venv
    \venv\Scripts\activate
    pip install pip --upgrade
    pip install -r requirements.txt
    ```
2. Create a .env file in web folder, the content will be like:
```
DEBUG=1
DJANGO_SUPERUSER_USERNAME=admin
DJANGO_SUPERUSER_PASSWORD=orange
DJANGO_SUERPUSER_EMAIL=641706106@qq.com
DJANGO_SECRET_KEY=fix_this_later

# if we test locally, then we ignore ssl
DB_IGNORE_SSL=true
POSTGRES_READY=0
POSTGRES_DB=dockerdc
POSTGRES_PASSWORD=mysecretpassword
POSTGRES_USER=myuser
POSTGRES_HOST=localhost
POSTGRES_PORT=5434

REDIS_HOST=redis_db
REDIS_PORT=6388

```
Make sure it's consistant with .yaml files and currently we don't need to configure postgres since it's just a local testing.
3. Run (in same folder of docker-compose.yaml)
`docker compose up --build` or run `docker compose up -d` to get in detached mode. If anything wrong, run `docker compose down` to remove what we built and build again

# Deploy on digitalOcean:
Since we've written github workflow file, we are albe to deploy our image on k8s when push project to github.

Go to digitalOcean, create a k8s cluster and follow the instruction to  connect it. To use image, we also need to create a container registry. After creating it, run `docker login registry.digitalocean.com`, the username and password will be digital ocean API token. Then go to github, settings--secrets and variables--actions--new repository secret, name is DO_API_TOKEN_KEY and value is API value of digital ocean. After configuration, whenever you push this project to github, it will automatically push image to container registry of digital ocean and create containers. You can also go to actions tab to check out each step.

# Deploy on digitalOcean manually:

## Push image:
Go to digitalOcean, create a k8s cluster and follow the instruction to  connect it. To use image, we also need to create a container registry. After creating it, run `docker login registry.digitalocean.com`, the username and password will be digital ocean API token. Then run 
```
docker build -t registry.digitalocean.com/my-cr/django-k8s:latest -f Dockerfile . 
docker push registry.digitalocean.com/my-cr/django-k8s --all-tags  
``` 
to build and push image to DigitalOcean.

## Configuration and deployment:
Create a database cluster on digital ocean and choose VPC to connect, then you are able to see all the credentials. Create a .env.prod file in web folder, copy the config of .env and configure postgres info according to credentials.

Now we run `kubectl.exe get secrets` and we are able to see the secret of digital ocean container registry.

Then we run `kubectl create secret generic django-k8s-web-prod-env --from-env-file=web/.env.prod` to create secret from .env.prod. If it's succesful, we are albe to use `kubectl.exe get secrets django-k8s-web-prod-env -o yaml` to see the detail of this secret.

After completing all these configuration, we are able to deploy our app by running `kubectl.exe apply -f .\django-k8s-web.yaml`. After executing it, we are able to see a new deployment by running `kubectl get deployments`. Since this deployment creates new pods and a new service, we can also check it by running `kubectl get services` and `kubectl get pods`.

To make sure whether we deploy it successful, we can enter pod by `kubectl exec -it [podName] -- bash`, then activate venv python by `source /opt/venv/bin/activate`. Afterward, run
```
import os
print(os.environ.get("DJANGO_SECRET_KEY"))
```
Now, we find that our venv python are able to use environment variable in .env.prod.

Additionally, we also need to configure allowed host. Since we already created a load balancer, run `kubectl.exe get services` to get EXTERNAL-IP of our service and put it in .env.prod. Since we change the .env.prod, we need to regenerate secret:
```
kubectl.exe delete secret django-k8s-web-prod-env
kubectl create secret generic django-k8s-web-prod-env --from-env-file=web/.env.prod
```