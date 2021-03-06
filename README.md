# distreg

Distreg is a distributed process registry.  Its main goal is to spread named worker processes (workers) evenly across an Erlang cluster using consistent hashing. It is designed to keep inter-node communication at a minimum. Only one instance of a worker identified by name is alowed to run.

Every worker is registered by its name and hashed to determine which node in the cluster it belongs to. Every node in the cluster keeps track of its own workers. Registering a worker on the local node is very fast and does not require any inter-node communication. When you need to call a worker on another node, distreg will determine which node it should be running on and call it on that node. 

Distreg will handle nodes leaving or joining as well as any conflicts. Whenever the cluster membership changes, distreg recalculates the consistent hashing ring and scans all local registered processes if they need to be moved to another node. If the node where a worker should be running has changed, the worker will be informed that it should restart itself on another node and it will be registered on the node where it should be running. Worker does not need to restart/die immediately, because both nodes will keep track of it. In the case of a conflict (more than one instance of a worker is running), all worker instances on the "wrong" nodes will be informed that they should die immediately and will be unregistered immediately as well.
Messages a worker might receive from distreg:
{distreg,shouldrestart}
{distreg,dienow}

Similar functionality exists in the global module. Global however only keeps multiple processes from being registered with the same name. It is also quite slow since it syncs the process table across all nodes for every registration. Gproc is another alternative, but it does not handle dynamic cluster reconfigurations and though it is much faster than global, it would still be much slower than distreg for the use case distreg was meant for.

Distreg does not need to be running on an erlang cluster. It can just be a very fast and simple local process registry. 

If a node in the cluster is not running distreg, it will be ignored.

## Build

```
$ rebar3 compile
```
