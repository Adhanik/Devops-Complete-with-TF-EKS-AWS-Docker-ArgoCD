

The output indicating that the architecture is `arm64` confirms that your Docker image was built for ARM architecture, while your EKS nodes are running on `amd64` architecture. This is the cause of the `exec format error` you're encountering. The solution is to rebuild your Docker image for `amd64` architecture.

### Steps to Fix the Issue

1. **Rebuild Docker Image for `amd64`**

   You need to explicitly specify the `linux/amd64` platform when building your Docker image. This ensures that the image is compatible with `amd64` architecture nodes in your EKS cluster.

   ```sh
   docker build --platform linux/amd64 -t adminnik/go-web-app:3 .
   ```

2. **Verify the Rebuilt Image**

   After rebuilding, verify that the architecture is now `amd64`:

   ```sh
   docker image inspect adminnik/go-web-app:v3 | grep Architecture
   ```

   The output should now indicate `amd64`.

   amitdhanik@Amits-MacBook-Air go-web-app % docker image inspect adminnik/go-web-app:v3 | grep Architecture

        "Architecture": "amd64",

3. **Push the Updated Image**

   Push the updated image to your container registry:

   ```sh
   docker push adminnik/go-web-app:v3
   ```

4. **Update Kubernetes Deployment**

   Ensure that your Kubernetes deployment is using the correct image. If you’ve updated the image tag, apply the changes to your Kubernetes cluster:

   ```sh
   kubectl set image deployment/go-web-app go-web-app=adminnik/go-web-app:v3
   ```

5. **Monitor the Pod**

   Check the status of the pod to ensure it’s no longer in a `CrashLoopBackOff` state:

   ```sh
   kubectl get pods
   ```

   If the pod is running, you should see its status as `Running` instead of `CrashLoopBackOff`.


amitdhanik@Amits-MacBook-Air go-web-app % kubectl get pods

NAME                          READY   STATUS    RESTARTS   AGE
go-web-app-6654f6598f-qwhvf   1/1     Running   0          9s


### Summary

- **Rebuild** your Docker image for the correct architecture using the `--platform linux/amd64` flag.
- **Verify** the architecture of the rebuilt image.
- **Push** the image to the container registry.
- **Update** your Kubernetes deployment with the new image.

By following these steps, you should resolve the architecture mismatch and get your pod running successfully on your EKS cluster.