# Annotations

| VNGCLOUD                                                       | Type                                        | Default                                  |
| -------------------------------------------------------------- | ------------------------------------------- | ---------------------------------------- |
| [/load-balancer-id](#load-balancer-id)                         | string                                      |                                          |
| [/load-balancer-name](#load-balancer-name)                     | string                                      | auto generate with rule                  |
| [/package-id](#package-id)                                     | string                                      | lbp-f562b658-0fd4-4fa6-9c57-c1a803ccbf86 |
| [/tags](#tags)                                                 | stringMap                                   |                                          |
| [/scheme](#scheme)                                             | internal / internet-facing                  | internal                                 |
| [/security-groups](#security-groups)                           | stringList                                  |                                          |
| [/inbound-cidrs](#inbound-cidrs)                               | string                                      | 0.0.0.0/0                                |
| [/healthy-threshold-count](#healthy-threshold-count)           | integer                                     | '3'                                      |
| [/unhealthy-threshold-count](#unhealthy-threshold-count)       | integer                                     | '3'                                      |
| [/healthcheck-interval-seconds](#healthcheck-interval-seconds) | integer                                     | '30'                                     |
| [/healthcheck-timeout-seconds](#healthcheck-timeout-seconds)   | integer                                     | '5'                                      |
| [/healthcheck-protocol](#healthcheck-protocol)                 | TCP / HTTP / HTTPS / PING-UDP               | TCP                                      |
| [/healthcheck-http-method](#healthcheck-http-method)           | GET / POST / PUT                            | GET                                      |
| [/healthcheck-path](#healthcheck-path)                         | string                                      | "/"                                      |
| [/healthcheck-http-version](#healthcheck-http-version)         | 1.0 / 1.1                                   | 1.0                                      |
| [/healthcheck-http-domain-name](#healthcheck-http-domain-name) | string                                      | ""                                       |
| [/healthcheck-port](#healthcheck-port)                         | integer                                     | traffic port                             |
| [/success-codes](#success-codes)                               | stringList                                  | '200'                                    |
| [/idle-timeout-client](#idle-timeout-client)                   | integer                                     | 50                                       |
| [/idle-timeout-member](#idle-timeout-member)                   | integer                                     | 50                                       |
| [/idle-timeout-connection](#idle-timeout-connection)           | integer                                     | 5                                        |
| [/pool-algorithm](#pool-algorithm)                             | ROUND_ROBIN / LEAST_CONNECTIONS / SOURCE_IP | ROUND_ROBIN                              |
| [/target-node-labels](#target-node-labels)                     | stringMap                                   | N/A                                      |

## Traffic Routing

Traffic Routing can be controlled with following annotations:

- <a name="load-balancer-id">`vks.vngcloud.vn/load-balancer-id`</a> specifies the id of the load balancer.

  > **⚠️ Warnings**: If you specify this annotation, load-balancer will not auto recreate when delete.
  >
  > **⚠️ Warnings**: If you want many ingress use a same load-balancer, we highly recommended use annotation [vks.vngcloud.vn/load-balancer-name](#load-balancer-name).

  ```yaml
  vks.vngcloud.vn/load-balancer-id: "lb-xxxxxxxxxxxxxx"
  ```

- <a name="load-balancer-name">`vks.vngcloud.vn/load-balancer-name`</a> specifies the custom name to use for the load balancer.

  > **ℹ️ Info**: Rule auto genearte load balancer name: **\*\***\*\***\*\***\_\_**\*\***\*\***\*\***
  >
  > **⚠️ Warnings**: Name longer than 50 characters will be treated as an error.
  >
  > **⚠️ Warnings**: Ingress with same this annotation value with use a same load-balancer.
  >
  > **⚠️ Warnings**: Update this field will cause create/update another load-balancer and redundant resource (old load-balabncer).

  ```yaml
  vks.vngcloud.vn/load-balancer-name: "custom-name"
  ```

- <a name="package-id">`vks.vngcloud.vn/package-id`</a> The ID of the network load-balancer package to be used for the service. If this annotation is not specified, the default package will be used.

  > **⚠️ Warnings**: Update this field after apply success will not effect.

  ```yaml
  vks.vngcloud.vn/package-id: "lbp-c531bc55-27d7-4a3e-be0b-eac265658a50"
  ```

- <a name="target-node-labels">`vks.vngcloud.vn/target-node-labels`</a> specifies which nodes to include in the target group registration.

  ```yaml
  vks.vngcloud.vn/target-node-labels: "worker=vmonitor,kubernetes.io/os=linux"
  vks.vngcloud.vn/target-node-labels: "key=v1,key=v2" # => "key=v2"
  ```

## Resource Tags

The VNGCLOUD Ingress Controller automatically applies following tags to the Load Balancer resources, it creates:

- `vks-cluster: ${clusterName}`

In addition, you can use annotations to specify additional tags

- <a name="tags">`vks.vngcloud.vn/tags`</a> specifies additional tags that will be applied to vLB resources created.

  > **⚠️ Warnings**: When user update tags manual in portal, our agent will not sync the change (load balancer not update `updateAt` when update tags)
  >
  > **⚠️ Warnings**: It'll update the config tag and append to the current tag.

  ```yaml
  vks.vngcloud.vn/tags: "Environment=dev,Team=test"
  ```

## Access control

Access control for LoadBalancer can be controlled with following annotations:

- <a name="scheme">`vks.vngcloud.vn/scheme`</a> specifies whether your LoadBalancer will be internet facing.

  ```yaml
  vks.vngcloud.vn/scheme: "internal"
  ```

- <a name="inbound-cidrs">`vks.vngcloud.vn/inbound-cidrs`</a> specifies the CIDRs that are allowed to access LoadBalancer.

  ```yaml
  vks.vngcloud.vn/inbound-cidrs: "10.0.0.0/24"
  ```

- <a name="security-groups">`vks.vngcloud.vn/security-groups`</a> specifies the securityGroups you want to attach to Node.

  > **⚠️ Warnings**: If you NOT specify this annotation, the controller will automatically create one security group, the security group will be attached to the Node and allow access from inbound-cidrs to the listen-ports.
  >
  > **⚠️ Warnings**: If you specify this annotation, you need to ensure the security groups on your Node to allow inbound traffic from the load balancer.
  >
  > **⚠️ Warnings**: If you specify this annotation, it'll configure only security group only include in this annotation. Ensure include them all here.

  ```yaml
  vks.vngcloud.vn/security-groups: "sg-xxxx,sg-yyyyy"
  ```

- <a name="idle-timeout-client">`vks.vngcloud.vn/idle-timeout-client`</a> Connection idle timeout is the maximum time a connection can remain open without any data transfer, after which the load balancer will close the connection. Range: (1-3600).

  ```yaml
  vks.vngcloud.vn/idle-timeout-client: "51"
  ```

- <a name="idle-timeout-member">`vks.vngcloud.vn/idle-timeout-member`</a> Backend member inactivity timeout in seconds. Range: (1-3600).

  ```yaml
  vks.vngcloud.vn/idle-timeout-member: "51"
  ```

- <a name="idle-timeout-connection">`vks.vngcloud.vn/idle-timeout-connection`</a> Backend member connection timeout in seconds.

  ```yaml
  vks.vngcloud.vn/idle-timeout-connection: "5"
  ```

## Health Check

Health check on target groups can be controlled with following annotations:

- <a name="healthcheck-port">`vks.vngcloud.vn/healthcheck-port`</a> specifies the port used when performing health check on targets.

  > **⚠️ Warnings**: The healthcheck port can automatically point to the protocol port.

- <a name="healthcheck-protocol">`vks.vngcloud.vn/healthcheck-protocol`</a> specifies the protocol used when performing health check on targets.

  ```yaml
  vks.vngcloud.vn/healthcheck-protocol: "HTTP"
  ```

- <a name="healthcheck-path">`vks.vngcloud.vn/healthcheck-path`</a> specifies the HTTP path when performing health check on targets.

  ```yaml
  vks.vngcloud.vn/healthcheck-path: "/ping"
  ```

- <a name="healthcheck-interval-seconds">`vks.vngcloud.vn/healthcheck-interval-seconds`</a> specifies the interval(in seconds) between health check of an individual target.

  ```yaml
  vks.vngcloud.vn/healthcheck-interval-seconds: '10'
  ```

- <a name="healthcheck-timeout-seconds">`vks.vngcloud.vn/healthcheck-timeout-seconds`</a> specifies the timeout(in seconds) during which no response from a target means a failed health check

  ```yaml
  vks.vngcloud.vn/healthcheck-timeout-seconds: '8'
  ```

- <a name="healthy-threshold-count">`vks.vngcloud.vn/healthy-threshold-count`</a> specifies the consecutive health checks successes required before considering an unhealthy target healthy.

  ```yaml
  vks.vngcloud.vn/healthy-threshold-count: '2'
  ```

- <a name="unhealthy-threshold-count">`vks.vngcloud.vn/unhealthy-threshold-count`</a> specifies the consecutive health check failures required before considering a target unhealthy.

  ```yaml
  vks.vngcloud.vn/unhealthy-threshold-count: '2'
  ```

- <a name="success-codes">`vks.vngcloud.vn/success-codes`</a> specifies the HTTP status code that should be expected when doing health checks against the specified health check path.

  ```yaml
  vks.vngcloud.vn/success-codes: "200,201"
  ```

- <a name="healthcheck-http-method">`vks.vngcloud.vn/healthcheck-http-method`</a> Define the HTTP method used for sending health check requests to the backend servers.

  > **⚠️ Warnings**: This option is applicable only when the [vks.vngcloud.vn/healthcheck-protocol](#healthcheck-protocol) is set to `http`.

  ```yaml
  vks.vngcloud.vn/healthcheck-http-method: "POST"
  ```

- <a name="healthcheck-http-version">`vks.vngcloud.vn/healthcheck-http-version`</a> Define the HTTP version used for sending health check requests to the backend servers.

  > **⚠️ Warnings**: This option is applicable only when the [vks.vngcloud.vn/healthcheck-protocol](#healthcheck-protocol) is set to `http`.

  ```yaml
  vks.vngcloud.vn/healthcheck-http-version: "1.1"
  ```

- <a name="healthcheck-http-domain-name">`vks.vngcloud.vn/healthcheck-http-domain-name`</a> The domain name, which be injected into the HTTP Host Header to the backend server for HTTP health check.

  > **⚠️ Warnings**: This option is applicable only when the [vks.vngcloud.vn/healthcheck-protocol](#healthcheck-protocol) is set to `http` and [vks.vngcloud.vn/healthcheck-http-version](#healthcheck-http-version) is set to `1.1`.

  ```yaml
  vks.vngcloud.vn/healthcheck-http-domain-name: "example.com"
  ```

## Pool configuration

- <a name="pool-algorithm">`vks.vngcloud.vn/pool-algorithm`</a> The load balancing algorithm used to determine which backend server to send a request to.

  ```yaml
  vks.vngcloud.vn/pool-algorithm: "SOURCE_IP"
  ```
