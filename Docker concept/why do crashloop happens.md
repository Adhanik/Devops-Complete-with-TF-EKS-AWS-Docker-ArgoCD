
# I created a Dockerfile, which you can see in GO-WEB-APP dir. We build the dockerfile, ran it, and tested it on our local machine and it was working fine(we were working with MAC)

Now we have a EKS cluster built on Amazon linux machine, and  when we created deployment using - kubectl apply -f k8s/manifests/deployment.yaml 

We got below -->

    kubectl get pods
    NAME                         READY   STATUS             RESTARTS      AGE
    go-web-app-9c6685f46-nrhrz   0/1     CrashLoopBackOff   3 (34s ago)   79s

how do I debug this?


### Sol

A `CrashLoopBackOff` status in Kubernetes indicates that a pod is starting, crashing, and then restarting repeatedly. To debug this, you'll need to gather more information about the pod to identify what’s causing the crashes. Here’s a step-by-step guide to troubleshoot this issue:

### 1. **Check Pod Logs**

The logs of the container within the pod often contain information about why the application is failing.

```sh
kubectl logs go-web-app-9c6685f46-nrhrz
```

If the pod has multiple containers, specify the container name:

```sh
kubectl logs go-web-app-9c6685f46-nrhrz -c <container-name>
```

### 2. **Describe the Pod**

The `kubectl describe` command provides detailed information about the pod, including events and reasons for failures:

```sh
kubectl describe pod go-web-app-9c6685f46-nrhrz
```

Look for:

- **Events Section**: Check for any warnings or errors related to the pod.
- **Container Status**: Check if there are any specific error messages or exit codes.

### 3. **Check Container Exit Codes**

Exit codes can help diagnose why a container is crashing. Common exit codes include:

- **0**: Success
- **1**: General error
- **137**: Container was terminated due to resource limits (e.g., out of memory)
- **143**: Container received a termination signal (SIGTERM)

If your container logs or `kubectl describe` shows an exit code, it can provide clues about the problem.

### 4. **Verify Resource Limits**

Check if the pod’s resource requests and limits are set appropriately. Resource limits that are too low can cause the container to be killed:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

If your application is exceeding these limits, it could cause crashes.

### 5. **Check Configuration and Environment Variables**

Ensure that all necessary configuration and environment variables are set correctly. Misconfigured environment variables or missing configuration can cause applications to fail:

```yaml
env:
  - name: DATABASE_URL
    value: "jdbc:mysql://db-service:3306/mydb"
```

Verify the values and correctness of these configurations in your pod specification.

### 6. **Inspect Deployment and ReplicaSet**

If the pod is part of a deployment or ReplicaSet, check their statuses as well:

- **Deployment**:
  ```sh
  kubectl get deployments
  kubectl describe deployment go-web-app
  ```

- **ReplicaSet**:
  ```sh
  kubectl get replicasets
  kubectl describe replicaset <replicaset-name>
  ```

Ensure that the deployment or ReplicaSet configuration is correct and there are no issues with scaling or rolling updates.

### 7. **Check for Application-Specific Issues**

If your application has specific logging or error handling, consult the application logs or documentation. Some applications might have internal logging that can help diagnose startup issues.

### 8. **Validate Container Image**

Ensure that the container image you are using is correct and properly built. Sometimes issues can arise from:

- **Corrupt Images**: Try pulling the image manually and running it outside Kubernetes.
- **Image Tags**: Make sure you're using the correct image tag.

### Example Workflow

Here’s a concise example workflow to diagnose the issue:

1. **Check Logs**:
   ```sh
   kubectl logs go-web-app-9c6685f46-nrhrz
   ```

2. **Describe Pod**:
   ```sh
   kubectl describe pod go-web-app-9c6685f46-nrhrz
   ```

3. **Check Resource Limits** (if applicable):
   Ensure your pod’s resource limits are adequate.

4. **Verify Configurations**:
   Check for correct environment variables and configurations.

5. **Inspect Related Kubernetes Objects**:
   Verify the status of deployments, ReplicaSets, or StatefulSets as applicable.

By following these steps, you should be able to identify the root cause of the `CrashLoopBackOff` and take appropriate actions to resolve it.