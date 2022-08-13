# Tips for debugging your Docker container

It is not a mystery when you containers crashes, and it can be a pain to figure out what the reason for the container crashing.

If you are stuck in this situation, here are some tips to help you as a start in your debugging process.

## check the logs
`docker logs container_id`\
This will give you the full STDOUT and STDERR from the command that was run initially in your container.

## check the stats
`docker stats container_id`\
Sometimes the container resources can hinder the container from running smoothly, the stats can give you a live stream of resource usage.

## attach your terminal to the container
`docker attach container_id`\
This is useful when you want to see what is written to STDOUT in real-time, or to control the process interactively using your terminal.
To detach from the container, use `CTRL-p` `CTRL-q` key combination.

## inspect the container
`docker inspect container_id`\
This views the container in details.
Some of the valuable information you might get from the inspection are:
* current state of the container (state)
* log path to see the log history (logpath)
* values of the environment variable (config.env)
* mapped ports (networksettings.port)

## check the history
`docker history container_id`\
This shows the history of the image to see how it was built

## shell into the container
`docker exec -it container_id sh`\
You can use exec to get an interactive shell in the container and start digging around manually

## override the ENTRYPOINT
Every docker image has an entrypoint and command, whether you define it or not in the dockerfile.
you can override the entrypoint to get access into the container and dig around.\
`docker run -d -p 80:80 --entrypoint /bin/sh image_name`
