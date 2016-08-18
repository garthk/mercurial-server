# mercurial-server

A [`mercurial-server`][ms] instance running under [`phusion/baseimage`][pb].

[ms]: http://www.lshift.net/work/open-source/mercurial-server/
[pb]: https://hub.docker.com/r/phusion/baseimage/

## Launch

If you've already had `mercurial-server` running:

    docker run -d -p 22 --name=hg -v=/var/lib/mercurial-server:/data garthk/mercurial-server

If not, and mounting data seems a terrible chore, create a `Dockerfile` and
`authorized_keys` quickly and build from those:

    cd `mktemp -d`
    echo FROM garthk/mercurial-server > Dockerfile
    cp ~/.ssh/id_rsa.pub authorized_keys
    docker build -t mercurial-server .
    docker run -d -p 22 --name=hg mercurial-server

Use the same trick to initialise a scratch `data` directory to mount: 

    cd `mktemp -d`
    echo FROM garthk/mercurial-server > Dockerfile
    cp ~/.ssh/id_rsa.pub authorized_keys
    docker build -t mercurial-server .
    CID=`docker run -d mercurial-server true`
    docker wait $CID
    docker cp $CID:data data
    docker rm $CID
    
... then, perhaps after copying `data` elsewhere:

    docker run -d -p 22 --name=hg -v=$PWD/data:/data garthk/mercurial-server

## Use

To discover your dynamic SSH port:

    docker port hg 22
    
Assuming `docker port` reported `0.0.0.0:32773`, try SSH as `hg`:

    $ ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no hg@0.0.0.0 -p 32773
    PTY allocation request failed on channel 0
    mercurial-server: direct logins on the hg account prohibited
    Connection to 0.0.0.0 closed.

If that worked, you can clone `hgadmin` and get cracking:

    hg clone --ssh='ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no' ssh://hg@0.0.0.0:32773/hgadmin

If you intend your instance to last a long time, leave out the SSH options.

## Build

To build the image, tagging it as `garthk/mercurial-server` so the usage examples above still work:

    docker build -t garthk/mercurial-server .
