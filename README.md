# quic_doh_docker
Traffic analyzer docker image for QUIC + DoH / TCP + TLS + DoH / etc.

# Requirements
Being a docker container, you have to have a running docker subsystem installed. If you have no such subsystem, first, go to [https://docs.docker.com/install/linux/docker-ce/debian/](https://docs.docker.com/install/linux/docker-ce/debian/), pick your distribution on the left hand side, and follow the instructions to install it.

It is also recommended to have `docker-compose` as it makes running the container much easier and transparent, not to mention the capability to run and manage multiple containers at once.


# Obtaining the container
Since dockerHub does no longer support automated builds for free users, you might build your image from scratch from the source to have an up-to-date version.
There might be an image up @dockerHub, but use it with cautions

## <a name="build"></a> Build your own image
Clone the repository first, then build the image.
```
git clone https://github.com/cslev/quic_doh_docker
cd doh_docker
sudo docker build -t cslev/doh_docker:latest -f Dockerfile  .
```
In the last command `-t` specifies the tag (default `latest`) used for our image! Feel free to use another tag, but to be sync with a possible future update might be coming from## Requirements

# Start container
## docker-compose
This is the easiest way to launch as the provided docker-compose.yml also shows you all details/variables you can set.
```
version: '3.3'

services:
  quic1:
    image: 'cslev/quic_doh_docker:latest'
    container_name: No_DoH  #rename according to you scenario for easier identification

    volumes:
      - './container_data/QUIC_DOH/archives:/quic_doh_project/archives:rw'
      #for PCAP files - default they are deleted due to storage concerns - set it below to keep them
      - './container_data/QUIC_DOH/pcap:/quic_doh_project/pcap:rw'
    dns: 1.1.1.1
    shm_size: '4g'

    environment:
      QUIC_DOH_DOCKER_NAME: 'QUIC_Docker'  #should be same as container_name
      QUIC_DOH_DOCKER_RESOLVER: ${R1} # resolver to use
      QUIC_DOH_DOCKER_START: '1' #first domain to visit in the domain list
      QUIC_DOH_DOCKER_END: '5000' #last domain to visit in the domain list
      QUIC_DOH_DOCKER_BATCH: '200' #how many domains to visit within a batch (don't change unless you know what you are doing)
      QUIC_DOH_DOCKER_INTF: 'eth0' #change this if your container would not have eth0 as a default interface
      QUIC_DOH_DOCKER_DOMAIN_LIST: 'top-1m.csv' #the relative path inside the container pointing to the list of domains to visit (in Alexa list style: id, domain <-one each line)
      QUIC_DOH_DOCKER_META: 'sg_quic_no_doh' #any meta info to affix your final output files for easier identification
      QUIC_DOH_DOCKER_WEBPAGE_TIMEOUT: '20' #how many seconds we wait for a website to load before throwing timeout error and skip
      QUIC_DOH_DOCKER_ARCHIVE_PATH: '/quic_doh_project/archives' #where to store the compressed output. Should be the same as defined by the 'volume'
      QUIC_DOH_DOCKER_USE_QUIC: '1' #set it to 1 to enable Quic or 0 to disable
      QUIC_DOH_DOCKER_KEEP_PCAP: '0' #DEBUG ONLY: pcap files can consume all your free space, Set it to 1 to keep pcap and 0 otherwise
```
One thing to note here is the variable `${R1}`. 
This is just to ease automation, you can freely use any of the DoH resolvers defined in the [r_config.json](https://github.com/cslev/quic_doh_docker/blob/main/source/r_config.json).

On the other hand, you can define variables in the `.env` (located in the same directory as your docker-compose yaml file), and use them in the yaml file.

The example above does the following:
 - visits the first 5000 domains in the Alexa's top 1M domains (provided by the `top-1m.csv` file in the source. 
 - Batch size of visiting and processing the data is set to 200 (it has been checked before several times about what the optimal number for batch size is, and 200 is the most efficient. Hence, it is not recommended to change this to a higher value; lower is fine :)
 - Firefox is enforced to try to use QUIC if possible
 - PCAP files will be deleted and only the analyzed .csv version of them will be kept. The .csv files are the same as the pcap with better resource usage and unnecessary information removed. You can keep the pcap files if needed, but bear in mind your data storage!
 - All data will be stored in the directories defined via the `volumes` tags
 - plain-text DNS server of the container is set to 1.1.1.1 for easier filtering

### Run via docker-compose
Issue the following command in the same directory, where your `docker-compose.yml` file is at.
```
$ sudo docker-compose up -d
```
Note, omit `-d` if you want to see the output.

### Process data
All data is found under the directory defined in the `volumes` tag.
Unzip the archive (and check the pcap with the SSL key if requested to store it), and process data.

## Docker
Define and pass all variables to the `docker run` command as it requires. Follow, the environment variables defined in the `docker-compose.yml` above.
For instance,
```
sudo docker run -d --name my_doh -e DOH_DOCKER_NAME=my_doh [-e FURTHER_ENV_VARS] -v <PATH_TO_YOUR_RESULTS_DIR>:/doh_project/archives:rw --shm-size 4g cslev/doh_docker:latest
```

