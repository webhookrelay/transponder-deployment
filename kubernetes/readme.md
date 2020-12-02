# Deployment

Prerequisites:

- Kubernetes cluster
- Postgres database (recommended to use a managed service)

## Creating configuration secrets

Admin credentials:

```
kubectl create secret generic admin-credentials --from-literal=username=<your username> --from-literal=password=<your password>
```

Create license secret:

```
kubectl create secret generic license --from-literal=license=<your license contents>
```

## Database and HTTPS settings

Edit database and HTTPS configuration sections in the deployment.yaml file. Transponder will be able to retrieve certificates from Let's Encrypt. For a database, we recommend using managed services such as AWS RDS or GCP CloudSQL. In both cases Postgres database should be used.

