%title: LEARN ANSIBLE
%author: xavki


# 47 : PE SWARM CLUSTER & TOOLS > install Traefik

<br>

>> subscribe to my channel <<


<br>

* create a specific network

```
- name: create traefik network
  docker_network:
    name: traefik_net
    driver: overlay
  when: inventory_hostname in groups['managers']
```

---------------------------------------------------------------------

# 47 : PE SWARM CLUSTER & TOOLS > install Traefik


<br>

* install traefik

```
- name: Deploy traefik service
  docker_swarm_service:
    name: traefik
    placement:
      constraints: "node.role==manager"
    publish:
    - { published_port: "80", target_port: "80" }
    - { published_port: "443", target_port: "443" }
    mounts:
      - source: /var/run/docker.sock
        target: /var/run/docker.sock
        type: bind
    read_only: yes
    restart_config:
      condition: any
      delay: 30s
      max_attempts: 5
    networks:
      - "traefik_net"
    image: "traefik:latest"
    args:
    - "--log.level=INFO"
    - "--api.dashboard=true"
    - "--entryPoints.http.address=:80"
    - "--entryPoints.api.address=:8080"
    - "--entryPoints.https.address=:443"
    - "--providers.docker.swarmmode"
    labels:
      traefik.enable: "true"
      traefik.swarmmode: "true"
      traefik.docker.network: "traefik_net"
      traefik.http.routers.traefik-public-http.rule: "Host(`traefik.swarm`)"
      traefik.http.routers.traefik-public-http.entrypoints: "http"
      traefik.http.routers.traefik-public-http.service: "api@internal"
      traefik.http.services.traefik-public.loadbalancer.server.port: "8080"
    replicas: 1
  when: inventory_hostname in groups['managers'][0]
```

---------------------------------------------------------------------

# 47 : PE SWARM CLUSTER & TOOLS > install Traefik


<br>

* install the visualizer

```
- name: install visualizer
  docker_swarm_service:
    name: visualizer
    image: dockersamples/visualizer
    #publish:
    #  - { published_port: 8080, target_port: 8080}
    networks:
      - "traefik_net"
    placement:
      constraints: "node.role==manager"
    mounts:
    - source: /var/run/docker.sock
      target: /var/run/docker.sock
      type: bind
    read_only: yes
    restart_config:
      condition: any
      delay: 30s
      max_attempts: 5
    labels:
      traefik.enable: "true"
      traefik.swarmmode: "true"
      traefik.docker.network: "traefik_net"
      traefik.http.routers.visu-public-http.rule: "Host(`visu.swarm`)"
      traefik.http.routers.visu-public-http.entrypoints: "http"
      traefik.http.services.visu-public.loadbalancer.server.port: "8080"
  when: inventory_hostname in groups['managers'][0]
```

