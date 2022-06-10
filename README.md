##### K8s Docker Registry with Redis caching

K8s Docker Registry with Redis for image caching

## Requirements
1) Spin up K8s cluster with 1 master ( control panel ) and 3 workers nodes.
2) Install helm and nginx ingress controller
3) Install container runtime ( docker )


## Setup

Install cert-manager:
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.0.2/cert-manager.yaml 

Install this docker-registry solution with Helm ( considering values.yaml is up to date ):
helm install docker-registry ./docker-registry
or
helm install docker-registry ./docker-registry \
--set registry.dns=registry.cmcloudlab782.info \
--set registry.email=ankit.asthana49@gmail.com 


Note: admin / admin1234 credentials should be changed for Production usage mentioned in values.yaml

## Local Setup

kubectl port-forward service/docker-registry 5000 -n docker-registry

         curl -u admin:admin1234 localhost.com:5000/v2/_catalog


## Production Setup

1) Define record set in Route53 register domain to point K8s cluster.
<img width="856" alt="Screenshot 2022-06-10 at 9 22 12" src="https://user-images.githubusercontent.com/59736927/173012477-90744673-3cd0-4211-baf8-d9a131130d7d.png">


2) cert-manager will automatically create a TLS certificate using Issuer and Certificate resources using Let's Encrypt

         curl -u user:password https://registry.cmcloudlab782.info/v2/_catalog


## Test

docker login https://registry.cmcloudlab782.info -u admin -p admin1234
docker pull busybox:latest
docker tag busybox:latest registry.cmcloudlab782.info/busybox:latest
docker push registry.cmcloudlab782.info/busybox:latest

## Clean Up

helm uninstall docker-registry


##### Additional Task

### Plan
Different docker image repositories need to be pruned based on different conditions (latest X images, images starting with PR, keep only semantic releases, etc) and needs to be automated.

### The Solution
1) Docker Registry Pruner image (https://github.com/tumblr/docker-registry-pruner) takes care of this by passing desired config file and docker-registry URL (See https://github.com/tumblr/docker-registry-pruner/blob/master/config/examples/example.yaml)
2) A Kubernetes CronJob will do the work by calling this image every 12 hours and prune all images matching desired configuration

### Considerations
1) Docker-registry image deletion needs to be enabled. This is already done by default in this solution.
2) If using AWS S3 for storage it's easier to do this by just using a clean up Lyfecicle Rule matching a particular object


### Improvements / Ideas
1) Added cert-manager for TLS support so it can be used in Production.
2) For the garbage collect cron job I've set a crontab that performs garbage-collect internally in registry container rather than using a Kubernetes CronJob.
3) AWS S3 is a better solution for storage as it's cheaper than Persistent Volumes.
4) Using Terraform and/or Ansible could be a potential next step for this soultion, in terms of automation.



### Contact
Ankit Asthana ( DevOps)

Project Link: https://github.com/ankitasthana-hu/private-docker-registry.git
