# Annotations

| VNGCLOUD                                                       | Type                                        | Default                                  |
| -------------------------------------------------------------- | ------------------------------------------- | ---------------------------------------- |
| [/ignore](#ignore)                                             | boolean                                     | false                                    |
| [/load-balancer-id](#load-balancer-id)                         | string                                      | ""                                       |
| [/load-balancer-name](#load-balancer-name)                     | string                                      | auto generate with rule                  |
| [/package-id](#package-id)                                     | string                                      | lbp-f562b658-0fd4-4fa6-9c57-c1a803ccbf86 |
| [/tags](#tags)                                                 | stringMap                                   | ""                                       |
| [/scheme](#scheme)                                             | internal / internet-facing                  | internet-facing                          |
| [/security-groups](#security-groups)                           | stringList                                  | auto create secgroup                     |
| [/inbound-cidrs](#inbound-cidrs)                               | string                                      | 0.0.0.0/0                                |
| [/healthy-threshold-count](#healthy-threshold-count)           | integer                                     | '3'                                      |
| [/unhealthy-threshold-count](#unhealthy-threshold-count)       | integer                                     | '3'                                      |
| [/healthcheck-interval-seconds](#healthcheck-interval-seconds) | integer                                     | '30'                                     |
| [/healthcheck-timeout-seconds](#healthcheck-timeout-seconds)   | integer                                     | '5'                                      |
| [/healthcheck-protocol](#healthcheck-protocol)                 | TCP / HTTP                                  | TCP                                      |
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
| [/enable-sticky-session](#enable-sticky-session)               | boolean                                     | false                                    |
| [/enable-tls-encryption](#enable-tls-encryption)               | boolean                                     | false                                    |
| [/target-node-labels](#target-node-labels)                     | stringMap                                   | ""                                       |
| [/certificate-ids](#certificate-ids)                           | stringList                                  | ""                                       |
| [/header](#header)                           | json                                  | "{"http":["X-Forwarded-For", "X-Forwarded-Proto", "X-Forwarded-Port"],"https":["X-Forwarded-For", "X-Forwarded-Proto", "X-Forwarded-Port"]}"                                       |
| [/client-certificate-id](#client-certificate-id) | string                                      | ""                                       |
| [/implementation-specific-params](#implementation-specific-params)                           | json                                  | "[]"                                       |

Compare with [AWS Ingress Annotation](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.7/guide/ingress/annotations/).

| AWS                                                                          | VNGCLOUD                                                                                                                                             | Type                    | Default                               | Location        | MergeBehavior |
| ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------- | ------------------------------------- | --------------- | ------------- |
| [/subnets](#subnets)                                                         | ❌ only subnet of worker node                                                                                                                        | stringList              | N/A                                   | Ingress         | Exclusive     |
| [/mutual-authentication](#mutual-authentication)                             | ❌ mutual authentication configuration                                                                                                               | json                    | '[{"port": 443, "mode": "off"}]'      | Ingress         | Exclusive     |
| [/certificate-arn](#certificate-arn)                                         | ❌ The first certificate in the list will be added as default certificate. And remaining certificate will be added to the optional certificate list  | stringList              | N/A                                   | Ingress         | Merge         |
| [/listen-ports](#listen-ports)                                               | ❌ set default                                                                                                                                       | json                    | '[{"HTTP": 80}]' / '[{"HTTPS": 443}]' | Ingress         | Merge         |
| [/group.name](#group.name)                                                   | ❌ IngressGroup resource                                                                                                                             | string                  | N/A                                   | Ingress         | N/A           |
| [/group.order](#group.order)                                                 | ❌ order across all Ingresses within IngressGroup. The smaller the order, the rule will be evaluated first                                           | integer                 | 0                                     | Ingress         | N/A           |
| [/load-balancer-attributes](#load-balancer-attributes)                       | ❌ access_logs.s3.enabled=true,...                                                                                                                   | stringMap               | N/A                                   | Ingress         | Exclusive     |
| [/ssl-redirect](#ssl-redirect)                                               | ❌ every HTTP listener will be configured with a default action which redirects to HTTPS                                                             | integer                 | N/A                                   | Ingress         | Exclusive     |
| [/target-type](#target-type)                                                 | ❌ specifies how to route traffic to pods (instance mode will route traffic to all ec2 instances, ip mode will route traffic directly to the pod IP) | instance / ip           | instance                              | Ingress,Service | N/A           |
| [/backend-protocol](#backend-protocol)                                       | ❌ Only support HTTP                                                                                                                                 | HTTP / HTTPS            | HTTP                                  | Ingress,Service | N/A           |
| [/target-group-attributes](#target-group-attributes)                         | ❌ specifies Target Group Attributes which should be applied to Target Groups                                                                        | stringMap               | N/A                                   | Ingress,Service | N/A           |
| [/ip-address-type](#ip-address-type)                                         | ❌❌ ipv4 / dualstack                                                                                                                                | ipv4 / dualstack        | ipv4                                  | Ingress         | Exclusive     |
| [/manage-backend-security-group-rules](#manage-backend-security-group-rules) | ❌❌ auto configure security group rules on Node/Pod                                                                                                 | boolean                 | N/A                                   | Ingress         | Exclusive     |
| [/customer-owned-ipv4-pool](#customer-owned-ipv4-pool)                       | ❌❌ specifies the customer-owned IPv4 address                                                                                                       | string                  | N/A                                   | Ingress         | Exclusive     |
| [/wafv2-acl-arn](#wafv2-acl-arn)                                             | ❌❌ specifies ARN for the Amazon WAFv2 web ACL                                                                                                      | string                  | N/A                                   | Ingress         | Exclusive     |
| [/waf-acl-id](#waf-acl-id)                                                   | ❌❌ specifies the identifier for the Amazon WAF web ACL                                                                                             | string                  | N/A                                   | Ingress         | Exclusive     |
| [/shield-advanced-protection](#shield-advanced-protection)                   | ❌❌ turns on / off the AWS Shield Advanced protection                                                                                               | boolean                 | N/A                                   | Ingress         | Exclusive     |
| [/ssl-policy](#ssl-policy)                                                   | ❌❌ specifies the Security Policy that should be assigned to the ALB                                                                                | string                  | ELBSecurityPolicy-2016-08             | Ingress         | Exclusive     |
| [/backend-protocol-version](#backend-protocol-version)                       | ❌❌ No options                                                                                                                                      | string                  | HTTP1                                 | Ingress,Service | N/A           |
| [/auth-type](#auth-type)                                                     | ❌❌ specifies the authentication type on targets                                                                                                    | none/oidc/cognito       | none                                  | Ingress,Service | N/A           |
| [/auth-idp-cognito](#auth-idp-cognito)                                       | ❌❌ specifies the cognito idp configuration                                                                                                         | json                    | N/A                                   | Ingress,Service | N/A           |
| [/auth-idp-oidc](#auth-idp-oidc)                                             | ❌❌ specifies the oidc idp configuration                                                                                                            | json                    | N/A                                   | Ingress,Service | N/A           |
| [/auth-on-unauthenticated-request](#auth-on-unauthenticated-request)         | ❌❌ specifies the behavior if the user is not authenticated                                                                                         | authenticate/allow/deny | authenticate                          | Ingress,Service | N/A           |
| [/auth-scope](#auth-scope)                                                   | ❌❌                                                                                                                                                 | string                  | openid                                | Ingress,Service | N/A           |
| [/auth-session-cookie](#auth-session-cookie)                                 | ❌❌                                                                                                                                                 | string                  | AWSELBAuthSessionCookie               | Ingress,Service | N/A           |
| [/auth-session-timeout](#auth-session-timeout)                               | ❌❌                                                                                                                                                 | integer                 | '604800'                              | Ingress,Service | N/A           |
| [/actions.${action-name}](#actions)                                          | ❌❌                                                                                                                                                 | json                    | N/A                                   | Ingress         | N/A           |
| [/conditions.${conditions-name}](#conditions)                                | ❌❌                                                                                                                                                 | json                    | N/A                                   | Ingress         | N/A           |

## Traffic Routing

Traffic Routing can be controlled with following annotations:

- <a name="ignore">`vks.vngcloud.vn/ignore`</a> specifies Ingress is ignored by controller.

  ```yaml
  vks.vngcloud.vn/ignore: "true"
  ```

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

- `vks-cluster-ids: ${clusterID}_${clusterID}_${clusterID}`

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

- <a name="certificate-ids">`vks.vngcloud.vn/certificate-ids`</a> specifies the certificates will be use in HTTPS listeners.

  > **⚠️ Warnings**: If you specify `.Spec.TLS` in your ingress resource, this annotation is requires to config certificate default for HTTPS listener.
  >
  > **⚠️ Warnings**: The first secret in list will be the default certificate, the other will in SNI cert list.

  ```yaml
  vks.vngcloud.vn/certificate-ids: "secret-xxx, secret-yyy"
  ```

- <a name="header">`vks.vngcloud.vn/header`</a> specifies the header for HTTP and HTTPS listeners.

  > **⚠️ Warnings**: If you specify in wrong format, the consequences will be unpredictable.
  >
  > **⚠️ Warnings**: You should choose the header allowed in portal.

  ```yaml
  vks.vngcloud.vn/header: "{"http":["X-Forwarded-For", "X-Forwarded-Proto", "X-Forwarded-Port"],"https":["X-Forwarded-For", "X-Forwarded-Proto", "X-Forwarded-Port","X-SSL-Client-Verify","X-SSL-Client-Has-Cert","X-SSL-Client-DN","X-SSL-Client-CN","X-SSL-Issuer","X-SSL-Client-SHA1","X-SSL-Client-Not-Before","X-SSL-Client-Not-After"]}"
  ```

- <a name="client-certificate-id">`vks.vngcloud.vn/client-certificate-id`</a> specifies the client certificate authentication will be use in HTTPS listener.

  ```yaml
  vks.vngcloud.vn/client-certificate-id: "secret-xxx"
  ```

- <a name="implementation-specific-params">`vks.vngcloud.vn/implementation-specific-params`</a> specifies the policy when use `ImplementSpecific` PathType. This annotation is an array of JSON objects, each object contains the path, rules (compare type `HOST_NAME`, `PATH`), and action (`REJECT`, `REDIRECT_TO_URL`, `REDIRECT_TO_POOL`).

  JSON format value:

  ```json
  [
    {
      "path": "/haha", // this value should match path value
      "rules": [
        {
          "type": "PATH", // HOST_NAME, PATH
          "compare": "EQUAL_TO", // CONTAINS, EQUAL_TO, REGEX, STARTS_WITH, ENDS_WITH
          "value": "/foo#" // value to compare
        },
        {
          "type": "PATH",
          "compare": "REGEX",
          "value": "/foo#anchor"
        }
        // more rules ...
      ],
      "action": {
        "action": "REJECT"                    // REJECT, REDIRECT_TO_URL, REDIRECT_TO_POOL
        "redirectUrl": "https://example.com", // required when action is REDIRECT_TO_URL
        "redirectHttpCode": 301,              // required when action is REDIRECT_TO_URL
        "keepQueryString": true               // required when action is REDIRECT_TO_URL
      }
    }
  ]
  ```
  
  > **⚠️ Warnings**: If you specify in wrong format, the consequences will be unpredictable.

  For example, when you have a rule use `ImplementSpecific` PathType, you can use this annotation to specify the policy for this rule.

  ```yaml
  spec:
    rules:
      - host: "a-1.vngcloud.vn"
        http:
          paths:
            - path: /haha # this value should match in annotation
              pathType: ImplementationSpecific
              backend:
                service:
                  name: netperf-service
                  port:
                    number: 80
  ```

  ```yaml
  # To create a policy REJECT for this rule, you can use this example:
  vks.vngcloud.vn/implementation-specific-params: '[{"path":"/haha","rules":[{"type":"PATH","compare":"EQUAL_TO","value":"/foo#"}],"action":{"action":"REJECT"}}]'

  # To create a policy REDIRECT_TO_URL for this rule, you can use this example:
  vks.vngcloud.vn/implementation-specific-params: '[{"path":"/haha","rules":[{"type":"PATH","compare":"EQUAL_TO","value":"/foo#"}],"action":{"action":"REDIRECT_TO_URL","redirectUrl":"https://example.com","redirectHttpCode":301,"keepQueryString":true}}]'

  # To create a policy REDIRECT_TO_POOL for this rule, you can use this example:
  vks.vngcloud.vn/implementation-specific-params: '[{"path":"/haha","rules":[{"type":"PATH","compare":"EQUAL_TO","value":"/foo#"}],"action":{"action":"REDIRECT_TO_POOL"}}]'
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

- <a name="enable-sticky-session">`vks.vngcloud.vn/enable-sticky-session`</a> Sticky session ensures that all requests from the user during the session are sent to the same target.

  ```yaml
  vks.vngcloud.vn/enable-sticky-session: "true"
  ```

- <a name="enable-tls-encryption">`vks.vngcloud.vn/enable-tls-encryption`</a> When true, connections to backend member servers will use TLS encryption.

  ```yaml
  vks.vngcloud.vn/enable-tls-encryption: "true"
  ```
