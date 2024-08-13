
# When we ran the kubectl log command, we got the below error 

amitdhanik@Amits-MacBook-Air go-web-app % kubectl logs go-web-app-9c6685f46-nrhrz
exec ./main: exec format error

# Sol

The error message exec format error typically indicates that the container image is not compatible with the architecture of the node where it's running. This usually happens when there's a mismatch between the architecture for which the Docker image was built and the architecture of the Kubernetes nodes (e.g., ARM vs. x86_64).

 **Inspect Docker Image**

   Ensure that the `adminnik/go-web-app:v2` image was built and pushed correctly. You can inspect the image metadata for details:

   ```sh
   docker image inspect adminnik/go-web-app:v2
   ```

   Check the `Architecture` field in the output to confirm it’s `amd64`.


So we checked our docker image that we build from dockerfile on mac, and we see that the architecture is arm64, while the EKS nodes are running on OS (Architecture) linux (amd64) with OS image Amazon Linux 2.

   docker image inspect adminnik/go-web-app:v2

       "Architecture": "arm64",
        "Variant": "v8",
        "Os": "linux",


1. **Docker Image Architecture**

   The `exec format error` often results from a mismatch between the architecture of the Docker image and the Kubernetes nodes. Building an image on a Mac could lead to this issue if the architecture isn't correctly specified.

   **Solution**: Explicitly specify the architecture when building your Docker image. Use Docker’s `--platform` flag to ensure the image is built for `linux/amd64`, which matches your EKS nodes.

   ```sh
   docker build --platform linux/amd64 -t adminnik/go-web-app:v2 .
   ```

So this caused our Issue.




