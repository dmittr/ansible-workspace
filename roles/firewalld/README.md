Example:

```
firewalld_default_zone: public
firewalld_interfaces:
  - { interface: "eth0", zone: "public" }

firewalld_sources:
  - { source: "192.168.1.0/24", zone: "internal" } 
  - { source: "203.0.113.50", zone: "drop" }

firewalld_services:
  - { service: "ssh", zone: "public" }
  - { service: "http", zone: "public" }
  - { service: "mysql", zone: "internal" }

firewalld_ports:
  - { port: 8080, proto: "tcp", zone: "public" }

firewalld_rich_rules:
  - { source: "192.168.1.100", port: 22, proto: "tcp", zone: "public", state: "enabled" }

firewalld_masquerade:
  - { zone: "public", state: "enabled" }

firewalld_forwarding:
  - { zone: "public", external_port: 8443, internal_port: 443, proto: "tcp" }

```