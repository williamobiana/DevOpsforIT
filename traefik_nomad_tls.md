# Running Traefik with TLS certificate on Nomad and Consul

We will be using Traefik as a reverse proxy for our nginx containers on port HTTP & HTTPS running in Nomad, while using Consul for service discovery.

## Prerequisites

Cluster running nomad (container orchestrator) and consul (service discovery & health checks)\
Basic understanding of traefik\
TLS certificate (self-signed or provided from a Certificate Authority)

## Structure

We have 2 set of applications running on Nomad (Traefik and Nginx).\
Traefik will communicate with consul for configurations and network information.\
Then traefik will forward the request to the nginx application

## traefik.nomad Job Spec

The traefik job of service type located in a defined datacenter had has a pod configuration
```
job "traefik" {
  type = "service"
  datacenters = ["dc1"]
  group "web" {}
}
```

The pod configuration for traefik with 1 instance
```
..
  group "web" {
    count = 1
    update {}
    network {}
    task "traefik" {}
  }
```

update configuration for traefik
```
..
  ..
    update {
      max_parallel = 1
      health_check = "checks"
    }
```

The network configuration that tells traefik what ports it must use and listen to.
by making it static, nomad will make local-host connect to the container on the same port 
```
..
  ..
    network {
      port "http" {
        to = 80
        static = 80
      }
      port "https" {
        to = 443
        static = 443
      }
      port "api" {
        to = 8080
        static = 8080
      }
    }
```

The traefik task defines the main activites the job will perform
```
..
  ..
    task "traefik" {
      driver = "docker"
      config {}
      template {}
      resources {}
      service {}
    }
```

The dockerfile prepares the container where our traefik service will run with its configurations.
* The image from docker hub
* The ports that should be open
* The arguments (information) to be passed to the contianer
* The volumes to be created 
```
..
  ..
    ..
      config {
        image = "traefik:v2.7"                
        ports = ["http", "https", "api"]
        arg = [
          "--configFile", "/etc/traefik/config.toml"
        ]
        volumes = [
          "traefik/config.toml:/etc/traefik/config.toml",
          "traefik/tls-certs.toml:/etc/traefik/tls-certs.toml",
          "certs/example.crt:/certs/example.crt",
          "certs/example.key:/certs/example.key",
        ]
      }
```

The traefik configuration template **traefik/config.toml** is written in toml, it can also be written with .yaml, --cli.flags and ENVIROMENT_VARIABLES.\
It creates the files (data) inside the container
* entrypoint to indicate how the 'network traffic' enters the container
* providers.file provides a path to the tls-certificate
* api for communicating with browser
* providers.consulCatalog attaches tags to the nodes that traefik will be giving 'network traffic'\
**Note: (consul and consulCatalog are not the same)**\
**Consul:** allow traefik to access consul kv store\
**ConsulCatalog:** exposes services that it tags
* logs and accessLogs 
```
..
  ..
    ..
      template {
        data = <<EOH
        
        [entryPoints]
          [entryPoints.http]
            address = ":80"
          [entryPoints.https]
            address = ":443"  
        
        [providers.file]
          filename = "/etc/traefik/tls-certs.toml"

        [api]
          insecure = true
          dashboard = true
        
        [providers.consulCatalog]
          exposedByDefault = true
          stale = true
          prefix = "traefik"
          constraints = "Tag(`traefik.tags=trk")
          [providers.consulCatalog.endpoint]
            address = "127.0.0.1:8500"

        [log]
          format = "json"
          level = "INFO"

        [accessLog]
          format = "json"

        EOH
        destination = "traefik/config.toml"
        change_mode = "restart"
      }
```

The **tls-certificate** template creates a file (data) in the container
* tls.certificate: path to the certifcate file and store them in the tls default store
* tls.store: path to the default store to pull the certificate file   
```
..
  ..
    ..
      template {
        data = <<EOH

        [[tls.certificate]]
          certFile = "/certs/example.crt"
          keyFile = "/certs/example.key"
          stores = ["default"]

        [tls.stores]
          [tls.stores.default]
            [tls.stores.default.defaultCertificate]
              certFile = "/certs/example.crt"
              keyFile = "/certs/example.key"

        EOH
        destination = "traefik/tls-certs.toml"
        change_mode = "restart"
      }
```

The **certs/example.crt** is the certifcate file
```
..
  ..
    ..
      template {
        data = <<EOH
-----BEGIN CERTIFICATE-----
MIID3zCCAsegAwIBAgIUOW8cnfw+...
-----END CERTIFICATE-----
        EOH
        destination = "certs/example.crt"
        change_mode = "restart"
      }
```

The **certs/example.key** is the public key file
```
..
  ..
    ..
      template {
        data = <<EOH
-----BEGIN PRIVATE KEY-----
MIIEvwIBADANBgkqhkiG9w0BAQEF...
-----END PRIVATE KEY-----
        EOH
        destination = "certs/example.key"
        change_mode = "restart"
      }
```

The resources for the node running the job
```
..
  ..
    ..
      resources {
        cpu = 512
        memory = 512
      }
```

The service configuration for the traefik deployment, for users to interact with the container
* based on the tags(rules & labels) assign to it.
* through a specific port 
* and perform health checks on the container(pod)
```
..
  ..
    ..
      service {
        name = "traefik-http"
        tags = []
        port = "http"
        check {
          type = "tcp"
          interval = "10s"
          timeout = "4s"
        }
      }
```

Put it all together, the complete traefik.nomad job spec should looks like this:
```
job "traefik" {
  type = "service"
  datacenters = ["dc1"]

  group "web" {
    count = 1

    update {
      max_parallel = 1
      health_check = "checks"
    }

    network {
      port "http" {
        to = 80
        static = 80
      }
      port "https" {
        to = 443
        static = 443
      }
      port "api" {
        to = 8080
        static = 8080
      }
    }

    task "traefik" {
      driver = "docker"
      config {
        image = "traefik:v2.7"
        ports = ["http", "https", "api"]
        arg = [
          "--configFile", "/etc/traefik/config.toml"
        ]
        volumes = [
          "traefik/config.toml:/etc/traefik/config.toml",
          "traefik/tls-certs.toml:/etc/traefik/tls-certs.toml",
          "certs/example.crt:/certs/example.crt",
          "certs/example.key:/certs/example.key",
        ]
      }

      template {
        data = <<EOH
        
        [entryPoints]
          [entryPoints.http]
            address = ":80"
          [entryPoints.https]
            address = ":443"  
        
        [providers.file]
          filename = "/etc/traefik/tls-certs.toml"

        [api]
          insecure = true
          dashboard = true
        
        [providers.consulCatalog]
          exposedByDefault = true
          stale = true
          prefix = "traefik"
          constraints = "Tag(`traefik.tags=trk")
          [providers.consulCatalog.endpoint]
            address = "127.0.0.1:8500"

        [log]
          format = "json"
          level = "INFO"

        [accessLog]
          format = "json"

        EOH
        destination = "traefik/config.toml"
        change_mode = "restart"
      }

      template {
        data = <<EOH

        [[tls.certificate]]
          certFile = "/certs/example.crt"
          keyFile = "/certs/example.key"
          stores = ["default"]

        [tls.stores]
          [tls.stores.default]
            [tls.stores.default.defaultCertificate]
              certFile = "/certs/example.crt"
              keyFile = "/certs/example.key"

        EOH
        destination = "traefik/tls-certs.toml"
        change_mode = "restart"
      }

      template {
        data = <<EOH
-----BEGIN CERTIFICATE-----
MIID3zCCAsegAwIBAgIUOW8cnfw+...
-----END CERTIFICATE-----
        EOH
        destination = "certs/example.crt"
        change_mode = "restart"
      }

      template {
        data = <<EOH
-----BEGIN PRIVATE KEY-----
MIIEvwIBADANBgkqhkiG9w0BAQEF...
-----END PRIVATE KEY-----
        EOH
        destination = "certs/example.key"
        change_mode = "restart"
      }

      resources {
        cpu = 512
        memory = 512
      }

      service {
        name = "traefik-http"
        tags = []
        port = "http"

        check {
          type = "tcp"
          interval = "10s"
          timeout = "4s"
        }
      }
    }
  }
}
```

## nginx.nomad Job Spec
```
job "nginx" {
  type = "service"
  datacenters = ["dc1"]

  group "web" {
    count = 1

    update {
      max_parallel = 1
      health_check = "checks"
    }

    network {
      port "http" {
        to = 80
        static = 80
      }

    task "nginx" {
      driver = "docker"

      config {
        image = "nginx"
        ports = ["http"]
        }

      resources {
        cpu = 512
        memory = 512
      }

      service {
        name = "nginx-http"
        tags = [
          "traefik.tags=trk",
          "traefik.http.routers.nginx.entrypoints=http",
          "traefik.http.routers.nginx.rule=PathPrefix(`/`)",
          "traefik.http.routers.nginx-secure.entrypoints=https",
          "traefik.http.routers.nginx-secure.rule=PathPrefix(`/`)",
          "traefik.http.routers.nginx-secure.tls=true",
        ]
        port = "http"

        check {
          type = "tcp"
          interval = "10s"
          timeout = "4s"
        }
      }
    }
  }
}
```

The service configuration for nginx service
* consul looks for services in nomad that ave health-checks and tags(rules & labels) defined on them
* the tag is divided into "prefix.protocol.traefik-service.appname.traefik-service-command"
```
..
  ..
    ..
      service {
        name = "nginx-http"
        tags = [
          "traefik.tags=trk",
          "traefik.http.routers.nginx.entrypoints=http",
          "traefik.http.routers.nginx.rule=PathPrefix(`/`)",
          "traefik.http.routers.nginx-secure.entrypoints=https",
          "traefik.http.routers.nginx-secure.rule=PathPrefix(`/`)",
          "traefik.http.routers.nginx-secure.tls=true",
        ]
        port = "http"

        check {
          type = "tcp"
          interval = "10s"
          timeout = "4s"
        }
      }
```     

## Troubleshooting

If a deplyment fails or you want to verify traefik deployed correctly.
Check for errors or review the steps in the logs 

check the status of the job\
`nomad status traefik`

check if the plan works before running\
`nomad plan traefik.nomad`\
`nomad run traefik.nomad`

read the status of the alloacated deployment\
`nomad alloc status "alloc_ID"`

read logs for the allocated deployment\
`nomad alloc logs "alloc_ID"`


## Summary

The expected state of your deployment should be a running nginx container on traefik ip address for 'http' and 'https'

There are many possible configuration for traefik, so feel free to test other alternatives
