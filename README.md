# Noded - A Slurm Node Monitoring Daemon

![Project Status](https://img.shields.io/badge/status-development-orange.svg)

## What is Noded?

Noded (pronounced node-d) is a python daemon that runs on compute nodes in a
Slurm HPC cluster.  Its purpose is to collect various system information and
job-related details for all Slurm jobs running on the compute node.

Existing monitoring daemons can monitor operating system metrics, protocol
metrics, and specific application metrics.  However, Slurm jobs in a cluster
typically make use of a variety of single and multi-threaded applications that
run on a single node or parallelized across multiple nodes. Most monitoring
daemons must be preconfigured with the process names to monitor ahead of time.
Noded will automatically discover all jobs running on the node.


## What do you need?

### supervisord

Slurm jobs finish very rapidly and if there are any uncaught exceptions, noded
dies.  With the supervisord watchdog, noded will respawn automatically, within
the thresholds configured in the supervisord configuration, for example no more
than 3 times in 60 seconds.


### Depends on Slurm configured with cgroups for CPU and or Memory:

* CR_CPU
* CR_CPU_Memory
* CR_Core
* CR_Core_Memory 


### Python 2.6
(need to test Python 2.7 and 3.3+)

Works on Linux only (depends on /proc)

Noded will only report jobs in the RUNNING state.  If a job is PENDING, it has
not be allocated resources on a node and Noded has no way of knowing about it.


## How does it work?

Noded periodically ships all job information on a Slurm node to a Redis
database in JSON format.

Noded looks for uids and jobids from the cgroup hierarchy, and then identifies
all children PIDs of a given job as well as the CPUs those processes are
running on.

Job data can be queried directly from the Redis database without having to
query the Slurm controller, adding load to slurmctld.

Redis was chosen because it is in-memory, very fast, and allows for a stateless
design.  On a cluster with 2000 nodes, 50,000 CPUs, and 5,000 concurrent jobs,
Redis consumes a mere 45 MB of RAM.  If Redis is down, all values are lost
since Redis is configured not to persist data.  This strategy works because as
Redis becomes available again, all nodes will repopulate the database within
the default 30 seconds.

Noded is single threaded.

## Installation

Requires python-redis and hi-redis for performance
Requires supervisord
Requires nose for testing

Requires a Redis server (>=2.8)


## Configuration

* nodename prefix length
* Redis credentials
* sleep timer
* redis key expiration
* overloaded noded ring buffer length
* load on by default
* enable memory, swap, network


## Starting/Stopping Noded

As root, use *supervisorctl* to start and stop noded, as well as to check the status:

```
# supervisorctl status
noded                            RUNNING   pid 44311, uptime 7 days, 3:24:43
```

```
# supervisorctl stop noded
noded: stopped
```

```
# supervisorctl start noded
noded: started
```

## Resources

* Schema Reference
* Examples
* Tuning Redis and OS


## Todo

* Capture more exceptions
* log exceptions with intuitive messages
* use exceptions as test cases
* provide help
* need to check if root is necessary, if not, provide instructions to create noded:noded
* provide ansible playbook for installation

## License

This project is in the worldwide public domain.  See
[LICENSE](LICENSE.MD) and [CONTRIBUTING](CONTRIBUTING.md) for more information.