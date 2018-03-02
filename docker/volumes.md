
# Volumes

## Managed Volume Permissions


### Using a Volume

When creating a a volume in the `Dockerfile`  by it default it will have `root` user permissions as seen below:

**Dockerfile**

```
ENV CONFLUENCE_TOMCAT_LOG_DIR "/opt/atlassian/confluence/logs"
VOLUME ["${CONFLUENCE_TOMCAT_LOG_DIR}"]
```

**Inspect running container**

Verify the volume has been created:

```
$ docker inspect confluence
...
            "Volumes": {
                "/opt/atlassian/confluence/logs": {},
                "/var/atlassian/application-data/confluence": {}
            },
...
```

Permissions are `root`:

```
$ ls -la /opt/atlassian/confluence
...
drwxr-x---.  2 root root   251 Feb 28 01:10 logs
...
```

### Set the Permission for the Volume

If we instead the set the permissions within the `Dockerfile` follows:

```
# Extract the installation directory and set the permission for the log directory
RUN mkdir -p                             ${CONFLUENCE_INSTALL_DIR} \
    && curl -L --silent                  ${DOWNLOAD_URL} | tar -xz --strip-components=1 -C "$CONFLUENCE_INSTALL_DIR" \
    && chown -R ${RUN_USER}:${RUN_GROUP} ${CONFLUENCE_INSTALL_DIR}/ \
    && chown -R ${RUN_USER}:${RUN_GROUP} ${CONFLUENCE_TOMCAT_LOG_DIR}/ \


VOLUME ["${CONFLUENCE_TOMCAT_LOG_DIR}"]    
```

**NOTE**:  
We deliberately put the `VOLUME` instruction after the `chown` operation because any modifications after the `VOLUME` instruction in a `Dockerfile` is not persisted. This is because we want the `chown` command to run on the image's filesystem but instead it will run it in the volume of a temporary container. By adding the `VOLUME` instruction as the end, docker will copy the files into the volume and set the right permissions.


```
$ $ ls -la /opt/atlassian/confluence
...
drwxr-x---.  2 daemon daemon   251 Feb 28 01:20 logs
...
```

### Host Bind Volume Mount

If we now do a host volume mount where `/tmp/logs` is not owned by the application user:

```
$ sudo docker run -v /tmp/logs:/opt/atlassian/confluence/logs --name="confluence" -d -p 8090:8090 -p 8091:8091 hniyaz/confluence
```

We will once again run into permission issues:

```
$ ls -la /opt/atlassian/confluence
...
drwxr-x---.  2 root root   251 Feb 28 01:10 logs
...
```

To get around this you will need to set the permissions on `/tmp/logs` on the host to allow the application user to have the correct access.


#### Portability 

Having to set the permissions on the host prior to starting the container makes it less portable as it relies on the deployment
system to ensure the permissions are already set.

To enable container portability and limit dependencies the solution is to set the permissions via the `entrypoint` script. This means you will need to start the container as `root` and then drop privileges within the `entrypoint` script.







