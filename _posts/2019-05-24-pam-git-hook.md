---
layout: post
title:  "Adding a git hook to a PAM 7.3 Installation"
date:   2019-05-24
categories: tech business-automation
---

# Adding a git hook to a PAM 7.3 Installation

These notes were taken from an enablement for DM/PAM 7.3 in April 2019.
The following steps will add a script to an existing Business Central (BC) install to
push commits within BC to a remote git repository. The script calls a java app
to integrate with the git repo. A later post may go into integrating more
directly. 

## post-commit hook script

Create a new directory for the script and associated app.
```
$ mkdir bc-git-hook
$ cd bc-git-hook
```

Download the app.
```
$ wget https://github.com/porcelli/bc-git-integration/releases/download/1.0.0-Beta1/bc-github-githook-1.0.0-Beta1.jar
```

Create a new file for the script and mark it executable.
```
$ touch post-commit
$ chmod a+x post-commit
```

Add this content to the file. Provide the Business Central base url for `BC_URL`
and Github username and access token for `GH_USERNAME` and `GH_PASSWORD`.
```
#!/bin/bash

BC_URL=
GH_USERNAME=
GH_PASSWORD=


java -jar \
    -Dsync.mode=on_sync \
    -Dbc.url=$BC_URL \
    -Dgh.username=$GH_USERNAME \
    -Dgh.password=$GH_PASSWORD \
    /opt/eap/standalone/data/kie/git/hooks/bc-github-githook-1.0.0-Beta1.jar
```

## Add script as git hook

Copy the script and jar to their own directory to be used for git hooks, ie. `/opt/eap/standalone/data/kie/git/hooks` or
`$JBOSS_HOME/standalone/data/kie/git/hooks`.

If running from you local EAP install, add this environment variable flag to the
boot command: `-Dorg.uberfire.nio.git.hooks=$PATH_TO_GIT_HOOKS_DIR`

If running from a local Docker container, add this to the `Dockerfile`

```
# Copy files for git hook                                                                                                  
RUN mkdir -p /opt/eap/standalone/data/kie/git/hooks                             
COPY support/bc-git-hook/ /opt/eap/standalone/data/kie/git/hooks/               
ENV org.uberfire.nio.git.hooks /opt/eap/standalone/data/kie/git/hooks 
```

If running in Openshift, add a pvc mounted to `/opt/eap/standalone/data/kie/git/hooks`, with working directory holding the script and jar

```
oc set volume dc/myapp-rhpamcentr --add --name=myapp-rhpamcentr-rhpamcentr-githook-pvol --claim-name=myapp-rhpamcentr-githook-claim --mount-path /opt/eap/standalone/data/kie/git/hooks --type pvc --claim-size=1G

# get pod name created after updating dc
oc get pods

oc rsync ./ myapp-rhpamcentr-2-$(NEW_POD_NAME):/opt/eap/standalone/data/kie/git/hooks/

# ensure files synced
oc rsh myapp-rhpamcentr-2-$(NEW_POD_NAME) ls -las /opt/eap/standalone/data/kie/git/hooks

# add env var
oc set env dc/myapp-rhpamcentr GIT_HOOKS_DIR=/opt/eap/standalone/data/kie/git/hooks
```

The `GIT_HOOKS_DIR` is specific to OpenShift images and templates. `org.uberfire.nio.git.hooks` is the
full environment variable. I lost a day figuring this out :grimace:

