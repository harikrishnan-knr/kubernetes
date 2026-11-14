docker compose build --no-cache frontend
docker compose up -d


I've written three Kubernetes files, but I haven't tested them on a cluster. Put them in a `k8s/` folder inside `Industrial-Project-Management/`.

- **`namespace.yaml`** creates a `wasteiq` namespace to keep everything together.
- **`backend.yaml`** has a Deployment (1 replica, port 8000, health checks, resource limits) and a Service named `backend`. The service name has to stay `backend`, because the nginx config in the frontend image forwards `/api/` to `http://backend:8000`.
- **`frontend.yaml`** has a Deployment (2 replicas, nginx on port 80) and a NodePort Service on port 30080 so you can open it in a browser.

**How to run it**

You need a local cluster such as minikube or kind, and the images must exist where the cluster can find them.

1. Build the images with the tags the manifests expect:
```bash
docker build -t wasteiq-backend:latest ./backend
docker build -t wasteiq-frontend:latest ./frontend
```

2. Load the images into your cluster. Use the line for the tool you have:
```bash
# minikube
minikube image load wasteiq-backend:latest
minikube image load wasteiq-frontend:latest

# kind
kind load docker-image wasteiq-backend:latest
kind load docker-image wasteiq-frontend:latest
```

3. Apply the manifests and check the pods:
```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/
kubectl get pods -n wasteiq
```

4. Open the app:
```bash
# minikube
minikube service frontend -n wasteiq

# kind or others
kubectl port-forward -n wasteiq svc/frontend 3000:80
```
Then go to http://localhost:3000.

**Things to know**
- **Backend replicas:** keep it at 1. The app uses SQLite, which is a file inside the container, so two backend pods would each have their own separate database.
- **Data loss on restart:** SQLite data resets whenever the backend pod restarts. For persistent data, either add a PersistentVolumeClaim or switch to Postgres.
- **Registry:** to run on a cloud cluster, push the images to a registry such as Docker Hub and change the `image:` lines to your registry names.
- **`ImagePullBackOff`:** if pods show this, the image wasn't loaded into the cluster (step 2). Run `kubectl describe pod <name> -n wasteiq` to confirm.

I can also write an Ingress file, or add Postgres with a persistent volume, if you want either.
