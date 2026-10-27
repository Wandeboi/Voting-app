# Example Voting App

A simple distributed application running across multiple Docker containers.

## Getting started

Download [Docker Desktop](https://www.docker.com/products/docker-desktop) for Mac or Windows. [Docker Compose](https://docs.docker.com/compose) will be automatically installed. On Linux, make sure you have the latest version of [Compose](https://docs.docker.com/compose/install/).

This solution uses Python, Node.js, .NET, with Redis for messaging and Postgres for storage.

Run in this directory to build and run the app:

```shell
docker compose up
```

The `vote` app will be running at [http://localhost:5000](http://localhost:5000), and the `results` will be at [http://localhost:5001](http://localhost:5001).

Alternately, if you want to run it on a [Docker Swarm](https://docs.docker.com/engine/swarm/), first make sure you have a swarm. If you don't, run:

```shell
docker swarm init
```

Once you have your swarm, in this directory run:

```shell
docker stack deploy --compose-file docker-stack.yml vote
```

## Run the app in Kubernetes

The folder k8s-specifications contains the YAML specifications of the Voting App's services.

Run the following command to create the deployments and services. Note it will create these resources in your current namespace (`default` if you haven't changed it.)

```shell
kubectl create -f k8s-specifications/
```

To remove them, run:

```shell
kubectl delete -f k8s-specifications/
```

## Architecture

![Architecture diagram](architecture.excalidraw.png)

* A front-end web app in [Python](/vote) which lets you vote between two options
* A [Redis](https://hub.docker.com/_/redis/) which collects new votes
* A [.NET](/worker/) worker which consumes votes and stores them in…
* A [Postgres](https://hub.docker.com/_/postgres/) database backed by a Docker volume
* A [Node.js](/result) web app which shows the results of the voting in real time

A voting app (vote, result, worker, redis, db) containerised, scanned, and deployed to Amazon EKS from code.

- **vote**: Python/Flask, port 5000
- **result**: Node.js, port 5000
- **worker**: .NET 8, moves votes from Redis to Postgres
- **redis**, **db** (Postgres 15)

## Run locally
    docker compose up -d --build
    vote: http://localhost:5000   result: http://localhost:5001

## Infrastructure (Terraform)
    cd terraform
    terraform init && terraform apply
    aws eks update-kubeconfig --name <cluster-name> --region eu-north-1
    kubectl get nodes
Teardown and rebuild: `terraform destroy`, then `terraform apply`.
State files are gitignored.

## CI/CD (GitHub Actions: .github/workflows/deploy.yml)
1. **test**: Compose smoke test
2. **build-scan-push**: builds each image, scans with Trivy (fails on HIGH/CRITICAL), pushes to Docker Hub tagged with the commit SHA
3. **deploy**: `helm upgrade --install` to EKS with that SHA

Repository secrets required: `DOCKERHUB_WANDE`, `DOCKERHUB_TOKEN`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`.

## Deploy with Helm
    helm upgrade --install voting ./chart --wait --timeout 5m \
      --set apps.vote.image.tag=<sha> \
      --set apps.result.image.tag=<sha> \
      --set apps.worker.image.tag=<sha>

## Security
- All app containers run as a non-root user (`appuser`)
- Images scanned by Trivy on every push
- Pods set `runAsNonRoot` and disable privilege escalation

## Teardown
    helm uninstall voting
    cd terraform && terraform destroy
Uninstall the chart first so the load balancers are deleted.

## Notes

The voting application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple
example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to
deal with them in Docker at a basic level.
