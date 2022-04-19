# 1.0 What is Envoy

The industry is moving toward microservice architectures and cloud-native solutions. With hundreds and thousands of microservices developed using different technologies, these systems can become complex and hard to debug.

As an application developer, you’re thinking about business logic – purchasing a product or generating an invoice. However, any business logic like that will result in multiple service calls between different services. Each of the services probably has its timeouts, retry logic and other network-specific code that might have to be tweaked or fine-tuned.

If at any point the initial request fails, it will be hard to trace it through multiple services, pinpoint where the failure occurred, and understand why it failed. Was the network unreliable? Do we need to adjust retries or timeouts? Or is it a business logic issue or a bug?

Adding to this complexity of debugging is the fact that services might use inconsistent tracing and logging mechanisms. These issues make it hard to identify the problem, where it happened, and how to fix it. This is especially true if you’re an application developer and debugging network issues falls outside of your core skills.

What would make debugging these network issues easier is to push networking concerns out of the applications stack and have another component deal with the networking part. This is what Envoy can do.

In one of its deployment patterns, we have an Envoy instance running next to every service instance. This type of deployment is also called a sidecar deployment. The other pattern Envoy works well with is the edge-proxy, which is used to build API gateways.

The Envoy and the application form an atomic entity but are still separate processes. The application deals with the business logic, and Envoy deals with network concerns.

In case of a failure, separating concerns makes it easier to determine if the failure is coming from the application or the network.
## Envoy: Out of process architecture

* Self-contained process
  * Sidecar model, transparent mesh
* Application don't need to worry about routing
* Works with any programming language
* Transparent deployment & upgrades

## Envoy: L3/L4 filter architecture

* L3/L4 = decision made based on IP addresses and TCP/UDP ports
* Puggable filter chain
  * Stack the filters to form a filter chain
* Existing filters: TC proxy, UDP proxy, HTTP proxy, TLS client Cert authentication
* Extensible!

## Envoy: L7 filter architecture

* HTTP filter layer
* HTTP connection management (HCM):
  * Buffering,
  * Rate limiting,
  * Routing/forwarding,

## Envoy: Firts-class HTTP/2 support

* Support for HTTP/1.1 and HTTP/2
* Any combination of HTTP/1.1 and HTTP/2 clientes and target servers can be bridged
* HTTP/2 between all Envorys = mesh of persistent connections

## Envoy: HTTP routing

* Routing subystem for routing and redirecting:
  * Path,
  * Authority
  * Content type
  * Runtime values
* Useful for front/edge proxy

## Envoy: gRPC ready

* gRPC: open-source RPC system that uses HTTP/2 for transport, protocol buffers as the IDL, and provides:
  * Authentication,
  * Bi-directional streaming,
  * Flow control, ...
* Envoy support all HTTP/2 features

## Envoy: Service discovery & dynamic config

* Static configuration (files)
* Dynamic configuration (xDS)
  * Configure Envoy through network

## Envoy: Health checking

* Passive & active healt checking
* Service discovery + health check information = determine healthy endpoints

## Envoy: Advanced load balancing

* Automatic retries
* Circuit breaking
* Global (and local) rete limiting
* Request shadowing/traffic mirrogin
* Outlier detection
* Request hedging

## Envoy: Front/edge proxy support

* HTTP/1.1, HTTP/2, and HTTP/3 support
* HTTP L7 routing
* TLS termination
  * TLS between all services! (mutual TLS)

## Envoy: Observability

* Logs, metrics, and traces
* Supports statsd (and compatible providers)
* Extensible!

## Envoy: HTTP/3 (Alpha)

* Support for HTTP/3
* Translation between HTTP/1.1, HTTP/2 and HTTP/3 in either direction!

# Envoy Build Blocks

The root of the Envoy configuration is called bootstrap configuration. It contains fields where we can provide the static or dynamic resources and high-level Envoy configuration (e.g., Envoy instance names, runtime configuration, enable the administrative interface, and so on).

To get started, we’ll mainly focus on the static resources, and later in the course, we’ll introduce how to configure dynamic resources.

Envoy outputs numerous statistics, depending on enabled components and their configuration. We’ll mention different stats throughout the course, and we’ll talk more about statistics in a dedicated module later in the course.

## Building blocks

* Bootstrap configuration
  * Static/dynamic resources & hight-level Envoy config
* Statistics for components

1. Listeners
2. Routes
3. Clusters
4. Endpoints

## Listeners

* Named network locations (IP address + port or Unix domain socket)
* Envoy receives request through listeners

```
static_rosources:
  listeners:
  - name: listener_0
    address:
      socket_address:
        address: 0.0.0.0
        port_value: 10000
    filter_chains: [{}]
```

## Network filters

* Operate on packet's payload
* Three categories:
  * Listener filters
  * Network filters
  * HTTP filters

## Routes

```
route_config:
  name: my_route_config
  virtual_hosts:
  - name: tetrate_host
    domains: ["tetrate.io"]
    route:
    ...
  - name: test_hosts
    domains: ["test.tetrate.io", "qa.tetrate.io"]
    routes:
    ...
```

## Routes: multiple domains

* Exact domain name (e.g. tetrate.io)
* Suffix domain wildcards (e.g. *.tetrate.io)
* Prefix domain wildcards (e.g. tetrate.*)
* Special wildcard matching any domain (*)

```
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address:
        address: 0.0.0.0
        port_value: 10000
    filter_chains:
    - filters:
      - name: envoy.filter.network.http_connection_manager
        type_config:
          "@type": type.googleapis.com/envoy.extensions.filter.network.http_connection_manager.v3.HttpConnectionManager
          stat_prefix: hello_world_service
          http_filters:
          - name: envoy.filter.http.router
          route_config:
            name: my_first_route
            virtual_hosts:
            - name: direct_response_service
              domains: ["*"]
              routes:
              - match:
                  prefix: "/"
                direct_response:
                  status: 200
                  body:
                    inline_string: "yay"
```
## Clusters

* Group of similar upstream hosts that accept the traffic:
  * List of hosts or IP addresses
* Multiple load-balancing algorithms
  * Round-robin
  * Least requested
  * Maglev, ...

## Endpoints

* Group of endpoints in a locality
* Wieghted endpoints for building hierachy within endpoints
* Locality for failover architecture

```
clusters:
- name: hello_world_service
  load_assigment:
    cluster_name: hello_world_service
    endpoints:
    - lb_endpoints:
      - endpoints:
          address:
            socket_address:
              address: 127.0.0.1
              port_value: 8000
```

## Optional features

* Active health checking
* Circuito breackers
* Outlier detection
* Protocol options for handling HTTP requests

## Obs: 

[Func-e makes running Envoy easy] {https://func-e.io/}

## Quiz: Envoy introduction

1. What are Envoy listeners? Named network locations
2. Envoy only works with application written in go = False
3. What's the name of the top-level element int the routing configuration?  Virtual host
4. Each request received on a listener can flow through a single filter = false
5. Envoy can be configuration through the network = true
6. What do we call a collection of centrally configured Envoys? Mesh
7. What are clusters? Group of similar upstream hosts that accept the traffic
8. Which filter is typically the last filter in the HTTP filter chain? Router filter
9. The envoy proxy and application form an atomic entity and are the same process = False
10. Which of the following are most typical Envoy deployment patterns?
    * Edge-proxy deployment
    * Sidecar deployment