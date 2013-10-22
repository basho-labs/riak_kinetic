riak_kinetic
============

This repository provides the Erlang files necessary for Riak to use Seagate
Kinetic Drives for key-value storage.

For more information about Seagate's Kinetic Drive, please see
[developer.seagate.com](http://developer.seagate.com/).

Installation
------------

These instructions assume you're willing to perform a source install of
Riak 1.4.2 from github. The instructions also assume you already have
Seagate's Kinetic Drive simulator, which you can get from
[github.com/Seagate/Kinetic-Preview](https://github.com/Seagate/Kinetic-Preview),
already running on `localhost` at port
8123.

These instructions also assume you've already installed Erlang,
[as explained here](http://docs.basho.com/riak/latest/ops/building/installing/erlang/).

Note that the `riak-install` script at the top level of this repository
contains all the installation commands, so feel free to execute the script
instead of copying and pasting the following commands.

First, clone this repository:

    git clone https://github.com/basho-labs/riak_kinetic.git

Next, clone `riak` and fetch all its dependencies:

    git clone https://github.com/basho/riak.git
    cd riak
    git checkout riak-1.4.2
    ./rebar get-deps

Next, install the `ekinetic` driver and dependencies as a dependency for
`riak_kv`:

    mkdir -p deps/ekinetic/ebin
    cp ../riak_kinetic/ebin/ekinetic* ../riak_kinetic/ebin/kinetic* deps/ekinetic/ebin
    ( cd deps/protobuffs ; git checkout master )
    perl -i -pe 's/(riak_pipe)/\1,ekinetic/' deps/riak_kv/src/riak_kv.app.src

Now, make a `riak` devrel release and install the
`riak_kv_ekinetic_backend` as shown below. Note that if you're running
Seagate's Kinetic Drive simulator on an endpoint other than `localhost`
port 8123, please adjust the `perl` command below appropriately.

    make devrel
    for i in {1..5} ; do
      cp ../riak_kinetic/ebin/riak_kv_ekinetic_backend.beam dev/dev$i/lib/basho-patches
      perl -i -pe 's/(storage_backend,\s*)[^}]+/\1riak_kv_ekinetic_backend/; print "{ekinetic, [{ip_port,{\"localhost\",8123}}]},\n" if (/%%\s*Bitcask/);' dev/dev$i/etc/app.config
    done

After this, please start `riak`
[as described here](http://docs.basho.com/riak/latest/quickstart/#Start-Up-Five-Nodes),
starting with the second step (`cd dev`). Note that if you ran the
`riak-install` script, you need to `cd riak/dev` instead before proceeding
with these steps.
