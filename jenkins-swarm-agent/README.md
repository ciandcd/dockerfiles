**[vipconsult/jenkins-swarm-agent](https://hub.docker.com/r/vipconsult/jenkins-swarm-agent)**  is simple image that runs the jenkins swarm plugin jar file which connects to a given master using parameters from  the Docker secret 

https://wiki.jenkins-ci.org/display/JENKINS/Swarm+Plugin

the only requirement is that the host has a docker daemon running and you set the jenkins master secret like
	
	echo "-master http://10.0.0.101 -password admin -username admin" | docker secret create jenkins-v1 -
* **10.0.0.101** replace it with actual Jenkins master IP
* *jenkins-v1 instead of just jenkins so that we can rotate the passwords - [Docker docs](https://docs.docker.com/engine/swarm/secrets/#example-rotate-a-secret)*



**On the testing docker swarm**

	docker service create \
		--mode=global \
		--name jenkins-swarm-agent \
		-e LABELS=docker-test \
		--mount "type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock" \
		--mount "type=bind,source=/tmp/,target=/tmp/" \
		--secret source=jenkins-v1,target=jenkins \
		vipconsult/jenkins-swarm-agent

**On the staging docker swarm**

	docker service create \
		--mode=global \
		--name jenkins-swarm-agent \
		-e LABELS=docker-stage \
		--mount "type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock" \
		--mount "type=bind,source=/tmp/,target=/tmp/" \
		--secret source=jenkins-v1,target=jenkins \
		vipconsult/jenkins-swarm-agent
	
**On the production Docker swarm**

	docker service create \
		--mode=global \
		--name jenkins-swarm-agent \
		-e LABELS=docker-prod \
		--mount "type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock" \
		--mount "type=bind,source=/tmp/,target=/tmp/" \
		--secret source=jenkins-v1,target=jenkins \
		vipconsult/jenkins-swarm-agent
	
* **mode=global** when we add new nodes to the Docker cluster they will all connect to the jenkins master to serve as slaves
* **mount "type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock"** we mount the Docker socket so that we can use the Docker client inside a container to start new containers or services directly on the host. This is a better approach than Docker in Docker
* **mount "type=bind,source=/tmp/,target=/tmp/"**  all containers can share files generated by the same Jenkins job
* **LABELS=docker-test**  the jenkins slave label. This is used when you define where to run each pipeline step - the labels need to match the ones used in the Jenkinsfile - node("docker-test") , node("docker-stage") , node("docker-prod")
* **secret source=jenkins-v1,target=jenkins** using the secret created in the previous step, needs to match a user on the jenkins master

Supported env variables - if not set the defaults are used

	-e LABELS="docker" - node labels
	-e EXECUTORS="3" - number of executors
	-e FSROOT="/tmp/jenkins" - data directory , jenkins need to have write access to - best to leave the defaults.
