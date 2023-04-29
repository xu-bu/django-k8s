# local deploying testing
1. Create python venv and install pkgs (open a cmd in root folder):
    ```
    python -m venv venv
    \venv\Scripts\activate
    pip install pip --upgrade
    pip install -r requirements.txt
    ```
2. run (in same folder of docker-compose.yaml)
`
docker compose up --build
`
or run `docker compose up -d` to get in detached mode. If anything wrong, run `docker compose down` to remove what we built and build again

3. `python manage.py runserver`

# Deploy on k8s:
Go to digitalOcean, follow the instruction to create a k8s cluster and connect it