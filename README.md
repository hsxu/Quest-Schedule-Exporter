# Install Tools

```
sudo apt-get install -y nodejs npm
sudo ln -s `which nodejs` /usr/bin/node

npm install -g gulp
npm install -g jshint
npm install -g browserify
```

# Local Setup


```
git clone git@github.com:Trinovantes/Quest-Schedule-Exporter.git
cd Quest-Schedule-Exporter
npm install
gulp debug
```

# Deploy to Production

These instructions are for deploying to a clean VPS host (e.g. DigitalOcean). This allows me to push changes to my website via the git command `git push prod master`.

First on the remote server:
```
mkdir questscheduleexporter.xyz.git && cd questscheduleexporter.xyz.git
git init --bare
```

Then create the `post-receive` hook:
```
cd hooks
touch post-receive
chmod +x post-receive
vim post-receive
```

Add the following to the `post-receive` file:
```
#!/bin/bash

GIT_REPO=$HOME/questscheduleexporter.xyz.git
TMP_GIT_CLONE=/tmp/questscheduleexporter.xyz
PUBLIC_WWW=/var/www/questscheduleexporter.xyz

git clone $GIT_REPO $TMP_GIT_CLONE
cd $TMP_GIT_CLONE

npm install
gulp build
rsync -a --delete $TMP_GIT_CLONE/dist/ $PUBLIC_WWW/

rm -Rf $TMP_GIT_CLONE
exit
```

Update the nginx configuration file (`/etc/nginx/sites-available/default`):
```
#-------------------------------------------------------------------------------
# questscheduleexporter.xyz
#-------------------------------------------------------------------------------

server {
    listen      80;
    server_name questscheduleexporter.xyz;
    return      301 https://www.questscheduleexporter.xyz$request_uri;
}

server {
    listen      80;
    server_name www.questscheduleexporter.xyz;
    autoindex   off;

    if ($request_uri ~ '/index.html') {
        rewrite ^ /$1 permanent;
    }

    location / {
        root /var/www/questscheduleexporter.xyz/;
    }
}
```

Finally on the local machine:
```
git remote add prod user@example.org:questscheduleexporter.xyz.git
```
