# Caesar Cipher Manifests

This repository contains Kubernetes manifests for deploying the Caesar Cipher application, designed to demonstrate basic encryption techniques using Kubernetes clusters.

For the source code and further details on the application itself, please refer to our separate [code repository](https://github.com/cicerowordb/caesar-cipher-code). 

The entire cluster is deployed on a single AWS instance running a lightweight K3S Kubernetes distribution, which is ideal for educational purposes and small-scale experiments like this one.

### Access and permissions

* Expose the TCP/5001 and TCP/6443 ports of your Instances using AWS security groups.

### Export your cluster config access

To export your Kubernetes cluster configuration for external access, use the following command sequence to replace the local IP with the public IPv4 address of your instance. This allows you to manage your cluster remotely via kubectl or other Kubernetes management tools.

```bash
INSTANCE_PUBLIC_IPV4=$(curl http://169.254.169.254/latest/meta-data/public-ipv4)
k3s kubectl config view --raw --output json \
    |tr -d " \n" \
    |sed "s/127.0.0.1/$INSTANCE_PUBLIC_IPV4/"
```

The first command gets the public IPv4 from your AWS instance and sets a new environment variable. The second command gets the K3S configuration and replaces the loopback IP for the instance public IP. The result is a JSON to be included inside a GitHub Actions secret: `secrets.KUBE_CONFIG`. Set this secret inside your repository before run the pipeline.

### Create new secret for Docker Hub private registry

To allow Kubernetes to pull images from a private Docker Hub registry, create a Kubernetes secret with your Docker credentials as shown below. This secret will be used by Kubernetes pods to authenticate to Docker Hub and enable the pulling of private images necessary for your deployment.

```bash
kubectl create secret docker-registry my-docker-credentials \
    --docker-server=https://index.docker.io/v1/ \
    --docker-username=<YOUR_DOCKER_USERNAME> \
    --docker-password=<YOUR_DOCKER_PASSWORD> \
    --docker-email=<YOUR_DOCKER_EMAIL>
```

# Deployment pipeline

This repository includes a continuous Deployment (CD) pipeline configured to trigger on pushes and pull requests to the main branch. Check the YAML file `.github/workflows/cd.yaml` to understand all steps.

The pipeline comprises several steps starting with the installation of `kubectl`, where a specific version is downloaded and set up on the system. Subsequent steps include exporting Kubernetes configuration from a GitHub secret into a `~/.kube/config` file to authenticate against a Kubernetes cluster.

The code is then checked out from the branch that triggered the action. The pipeline performs local and remote dry runs of Kubernetes configurations to validate them before finally applying the configurations to the cluster. This ensures that each change pushed to the main branch is automatically tested for syntax and feasibility in the cluster environment before deployment, enhancing reliability and automation in the deployment process.

### Known issues

The `--insecure-skip-tls-verify` option is used with kubectl to instructs the tool to bypass the verification of TLS certificates. This is necessary because it K3S installation uses the private and loopback IPs in the certificate (by default) and we are using the public IP to access the API. To avoid additional steps to explaing how to replace the K3S API certificate, we are simply ignoring certificate issues.
