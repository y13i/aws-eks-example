# AWS EKS Example

This is a **shortened version** of [Get Started](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html).

- No manual operation from the Management Console.
- CFn stacks merged into one.
- 2 public subnets and 2 private subnets (with NAT gateways).
- Worker nodes launches into private subnets.
- Renamed some files and resources.

## Install tools

1.  Download [`kubectl` and `heptio-authenticator-aws`](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html#get-started-kubectl) to current directory.
2.  Upgrade to latest AWS CLI.

## Deploy stack

```sh
$ aws cloudformation deploy --template-file cfn.yml --capabilities CAPABILITY_IAM --stack-name eks-example
```

## Configure kubectl

```sh
$ aws cloudformation describe-stacks --stack-name eks-example --query Stacks[0].Outputs
```

Edit `k8s-config.yml` and `k8s-configmap-aws.yml`.

Then, set `KUBECONFIG`.

```sh
$ export KUBECONFIG=k8s-config.yml
```

## Join nodes to the cluster

```sh
$ ./kubectl apply -f k8s-configmap-aws.yml
```

```sh
$ ./kubectl get nodes
NAME                                       STATUS    ROLES     AGE       VERSION
ip-10-0-12-27.us-west-2.compute.internal   Ready     <none>    26s       v1.10.3
ip-10-0-9-187.us-west-2.compute.internal   Ready     <none>    29s       v1.10.3
```

Cool.

## Deploy some sample apps

For instance, [WordPress](https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/).

### Define storage class

Before claiming a persistent volume, you need to define an EBS storage class and set it default.

```sh
$ ./kubectl apply -f k8s-storageclass-ebs-gp2.yml
$ ./kubectl get storageclass
NAME            PROVISIONER             AGE
gp2 (default)   kubernetes.io/aws-ebs   13s
```

[Storage Classes - Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/storage-classes.html)

### Deploy

```sh
$ ./kubectl create secret generic mysql-pass --from-literal=password=THEMOSTSECUREPASSWORDEVER
$ ./kubectl create -f https://github.com/kubernetes/website/raw/master/content/en/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/mysql-deployment.yaml
```

This operation creates an EBS volume for persistent volume.

```sh
$ ./kubectl create -f https://github.com/kubernetes/website/raw/master/content/en/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/wordpress-deployment.yaml
```

And this operation creates a load balancer.

```sh
$ ./kubectl describe services wordpress
```

Ta-da!

You can access the load balancer's DNS name and you'll see the WP initial setting page!

## Clean up

```sh
$ ./kubectl delete secret mysql-pass
$ ./kubectl delete deployment -l app=wordpress
$ ./kubectl delete service -l app=wordpress
$ ./kubectl delete pvc -l app=wordpress
$ aws cloudformation delete-stack --stack-name eks-example
```
