# Updating Resources Accordingly

## Updating Images

You can migrate to vCR.

## Updating Services

After the cluster is migrated, maybe your VKS cluster will create new LoadBalancer from vLB if source cluster have resource Service type LoadBalancer. You can check whether the Service is available.

## Updating Ingress

As same as Service type LoadBalancer, the VKS cluster has integrated ingress controller support from vLB layer 7. You have to config `ingressClassName: vngcloud` to use this feature.

## Updating the Storage Class

You have to migrate all `StorageClass` in source cluster to vks cluster. You can reuse the name with our provider.

## Updating Databases
