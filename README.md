# Docker deployment orchestration for Vaadin CRM (React)

This repo contains a docker-compose script to simulate the demo.vaadin.com
container orchestration system and test that the docker images for
[the REST API backend](https://github.com/vaadin/crm-react-backend) and for
[the React Web UI](https://github.com/vaadin/crm-react) are built and configured
correctly.

## Clone the repo and run `docker-compose` locally
1. (initial) `git clone --recurse-submodules https://github.com/vlukashov/crm-react-deployment`
1. (update) `git pull && git submodule update --remote --recursive`
1. create production builds both of the API backend and the React Web UI app (see the instructions in the README files in `./crm-react-backend` and `./crm-react` respectively)

   NOTE: Running the build locally requires having a local development environment both for Spring Boot and for React, i.e. JDK, Maven, Node and Yarn. Running the build inside a container is not supported at the moment.
1. build docker images both for the API backend and the React Web UI app, and for the reverse proxy that routes browser requests either to the API or to the Web UI: `docker-compose build`
1. start all 3 containers: `docker-compose up`
1. check that the app is running on [http://localhost:8080](http://localhost:8080)
1. stop all containers: `Ctrl+C`
1. delete the container images: `docker-compose down`

## Proxy

The `proxy` container acts as a client-facing reverse proxy.
It accepts all network requests and proxies them either to the React Web UI, or to the REST API backend.
The routing is based on the request URL: all requests to `/api/**` are routed to the REST API, everything else is routed to the React Web UI.
This is done primarily to keep a 1-to-1 mapping between the source repos and deployed containers, and be able to deploy and restart them separately.

Should you need to change the `/api` URL to something else, this should be done in several places simultaneously:
 - in the React Web app source code (see `./crm-react/.env.production`)
 - in the proxy image's NginX config in `./crm-react-proxy/nginx.conf` in this repo
 - in the docker-compose file in this repo

This is not ideal, but making this a single config property requires passing configuration to the React Web UI through run-time environment variables. It is [non-trivial](https://www.freecodecamp.org/news/how-to-implement-runtime-environment-variables-with-create-react-app-docker-and-nginx-7f9d42a91d70/) and has not been done.

## Troubleshooting

In order to run an interactive shell in one of the running containers execute `docker exec -it [container-name] /bin/ash` (e.g. `docker exec -it crm-react-deployment_proxy_1 /bin/ash`).

The names of the individual containers can be found in the `docker-compose up` output, or by running `docker ps`.