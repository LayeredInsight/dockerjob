# dockerjob
[![Docker Pulls](https://img.shields.io/docker/pulls/layeredinsight/dockerjob.svg?style=plastic)](https://hub.docker.com/r/layeredinsight/dockerjob/)
*dockerjob* arose from a need to be able to easily schedule backup
jobs in our VM-image-based software distributions. So we tracked down [Crocker](https://github.com/APNIC-net/crocker/) - the cool, light-weight cron-like scheduler desgined for containers, and paired it with [Docker's](https://docker.com)...[docker](https://hub.docker.com/_/docker/).

The result? The abilty to run a container that runs another container at a specified time, be that for backups, wakeup calls, chaos injection, or whatever other automatable containerism comes to mind...

# Usage
To use docker inside the *dockerjob* container, you'll need to bind-mount your docker socket into the container. From a command line, this is done by providing the parameters `-v /var/run/docker.sock:/var/run/docker.sock`, presuming your docker socket is in the default location.

*dockerjob* could be run from the commandline...
```
docker run --rm -ti -v /var/run/docker.sock:/var/run/docker.sock --entrypoint /usr/local/bin/crocker dockerjob -A "09 10 * * *" docker ps
```

...but really we wrote this for repeatability and ease-of-use. In a `docker-compose.yml` file, it could be used like so:

```yaml
  version: "3.2"
  services:
    mongodb:
      image: mongo:3.2.4
      ports:
        - "27017:27017"
      volumes:
        - type: volume
          source: /data/mongo-data
          target: /data/db
    mongo-backup:
      image: layeredinsight/dockerjob:1.0
      entrypoint: /usr/local/bin/crocker @daily docker run --rm -ti -v /data/mongo-backups:/data/backups mongo:3.2.4 mongodump --gzip -o /data/backups/`date +%Y%m%d` --host=mongodb
      volumes:
        - type: bind
          source: /var/run/docker.sock
          target: /var/run/docker.sock
```

More details about the arguments to `/usr/local/bin/crocker` can be found on the [crocker github page](https://github.com/APNIC-net/crocker/).

For more details on the format of the cronjob time expression, see the [vixiecron manpage(5)](https://linux.die.net/man/5/crontab).

## Troubleshooting
You need to bind mount the docker socket inside the *dockerjob* container, otherwise you will see the following when trying to run docker commands:

> Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?

## One Final Note...
*dockerjob* is not what the sample says in Fatboy Slim's [Soul Surfing](https://www.youtube.com/watch?v=cEkWjE6ez00).
