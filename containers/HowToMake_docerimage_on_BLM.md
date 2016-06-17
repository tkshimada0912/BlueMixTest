# How to make original container running in BlueMix environment and first impression

This is the way to make original container environment running on BlueMix container environment.

## preparation

You need cf / ci CLI interfae for CloudFoundry

- cf commandline tool https://github.com/cloudfoundry/cli/releases
- ci plugin https://console.ng.bluemix.net/docs/containers/container_cli_ov.html#container_cli_ov

check environment whether all packages is installed.

    % cf plugins
    % cf ic

## create local image

First, you should make docker image in local environment.
I used "Kitematic" docker environment on local Mac OS X environment.
It can be able to make image on any environment which docker runs.

### make docker file

I created ssh enabled container for environment checking.

Dockerfile example:

    FROM centos:centos7
    RUN yum install -y openssh-server openssh-clients
    RUN mkdir /root/.ssh
    RUN echo "ssh-dss AAA.....XXXXXX shimada@shimadabu-no-MacBook-Pro.local" > /root/.ssh/authorized_keys
    RUN chmod 700 /root/.ssh
    RUN chmod 600 /root/.ssh/authorized_keys
    RUN /usr/bin/ssh-keygen -q -t rsa -f /etc/ssh/ssh_host_rsa_key -N ''
    EXPOSE 22
    CMD /usr/sbin/sshd -D

### docker build

    # docker build -t sshtest .

The docker image named 'sshtest' was created.
check the output of "docker images" command.

### test run locally

    # docker run -d -n TEST sshtest

container is up.
login and check environment.

    # docker exec -it TEST /bin/bash
    # ssh localhost

You shuld setup ssh(private key) collectly to RSA key login.

### tag and push

assign tag to image and push to BlueMix

    # docker tag sshtest registry.ng.bluemix.net/tk_shimada/sshtest:test

Before push image to repository you must logged into BlueMix with CLI.

    # cf login
    # cf ic login

check the environment.

    # cf ic ps

No container is running. It is good.

Then push image to BlueMix repository.

    # docker push "registry.ng.bluemix.net/tk_shimada/centos:test"

## run container in BlueMix environment

    # cf ic run --name TEST2 -p 22 registry.ng.bluemix.net/tk_shimada/sshtest:test

status is following.

    CONTAINER ID        IMAGE                                             COMMAND             CREATED             STATUS                   PORTS               NAMES
    e95d0397-ec0        registry.ng.bluemix.net/tk_shimada/sshtest:test   ""                  53 seconds ago      Running 24 seconds ago   22/tcp              TEST2

- CONTAINER ID is not fully displayed. full name is 'e95d0397-ec0c-46f7-9ba7-2937fa87d128'. You can use 'cf ic inspect' command to check detail status.
- COMMAND is NULL, but the command applied in Dockerfile is running...
- PORT is locally forwarded. Must to bind PUBLIC ip address manually.

## add IP address

Assign PUBLIC ip address to container for access from Internet.

    # cf ic ip list
    # cf ic ip request
    # cf ip bind "169.44.119.20" TEST2

if IP is not allocated, you should request IP address assignment.
Then you can bind address to container.
if assignment success, container status is updated like following.

    # cf ic ps
    CONTAINER ID        IMAGE                                             COMMAND             CREATED             STATUS                  PORTS                      NAMES
    e95d0397-ec0        registry.ng.bluemix.net/tk_shimada/sshtest:test   ""                  9 minutes ago       Running 8 minutes ago   169.44.119.20:22->22/tcp   TEST2

ping is OK, but it takes some seconds to enable ip address.

    # ping 169.44.119.20
    PING 169.44.119.20 (169.44.119.20): 56 data bytes
    64 bytes from 169.44.119.20: icmp_seq=0 ttl=50 time=153.062 ms
    64 bytes from 169.44.119.20: icmp_seq=1 ttl=50 time=152.452 ms

## check login

    ssh -i ~/.ssh/id_dsa 169.44.119.20 -l root

You can logged into container environment with public-key authentication.

## BlueMix coutainer environment

I checked container environment inside.
BlueMix container environment looks very standard configuration.

- assigned ip address is '172.31.0.2/16', this looks standard bridge-based docker environmnet.
- GW is 172.31.0.1, this looks fixed assigned docker host configuration. it is major configuration.
- CPU: E5-2690 / 48 processors 2 physical,12 cores,2 threads
- 256GB memory
- kernel 3.19.0-49-generic #55~14.04.1-Ubuntu
- root filesystem size is 9.8GB. This is standard I thnk.

it looks like very natural, normal, common configuration.

But there's just a little difference in global/public IP address assignment.
Normally docker containers in a docker host shares outbound interface address, and 
expose any service endpoint with tcp port number.
In Bluemix container environment, containers hold discrete ip address.
It is enable to do these configuration with any docker network driver.
But it is not a normal configuration, it may need any paticular setting in docker host.

BlueMix cf ic command API is just a little unpreasant.

- response is slow
- sometime no response from repository
- slow to bind ip address

Container environment is expected to be deployed at high speed, I think.
But BlueMix of container environment feel a little unsatisfactory.
