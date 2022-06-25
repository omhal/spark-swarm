# SPARK SWARM
Software and tools for setting up and operating a personal compute cluster, with focus on big data.

install local registry

docker pull registry-1.docker.io/distribution/registry:2.1

docker run -d -p 5000:5000 --restart=always --name master registry:2
