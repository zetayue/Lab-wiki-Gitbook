# Submitting Docker Job

## Introduction of Docker Universe

A docker universe job instantiates a Docker container from a Docker image, and HTCondor manages the running of that container as an HTCondor job, on an execute machine. This running container can then be managed as any HTCondor job. For example, it can be scheduled, removed, put on hold, or be part of a workflow managed by DAGMan.

The docker universe job will only be matched with an execute host that advertises its capability to run docker universe jobs. When an execute machine with docker support starts, the machine checks to see if the _docker_ command is available and has the correct settings for HTCondor. Docker support is advertised if available and if it has the correct settings.

The image from which the container is instantiated is defined by specifying a Docker image with the submit command `docker_image`. This image must be pre-staged on a docker hub that the execute machine can access.

After submission, the job is treated much the same way as a vanilla universe job. Details of file transfer are the same as applied to the vanilla universe. One of the benefits of Docker containers is the file system isolation they provide. Each container has a distinct file system, from the root on down, and this file system is completely independent of the file system on the host machine. The container does not share a file system with either the execute host or the submit host, with the exception of the scratch directory, which is volume mounted to the host, and is the initial working directory of the job. 

In Docker universe \(as well as vanilla\), HTCondor never allows a containerized process to run as root inside the container, it always runs as a non-root user. It will run as the same non-root user that a vanilla job will. If a Docker Universe job fails in an obscure way, but runs fine in a docker container on a desktop, try running the job as a non-root user on the desktop to try to duplicate the problem.

HTCondor creates a per-job scratch directory on the execute machine, transfers any input files to that directory, bind-mounts that directory to a directory of the same name inside the container, and sets the IWD of the contained job to that directory. The assumption is that the job will look in the cwd for input files, and drop output files in the same directory. In docker terms, we docker run with the `-v /some_scratch_directory -w /some_scratch_directory -user non-root-user` command line options \(along with many others\).

The executable file can come from one of two places: either from within the container’s image, or it can be a script transfered from the submit machine to the scratch directory of the execute machine. To specify the former, use an absolute path \(starting with a /\) for the executable. For the latter, use a relative path.

Therefore, the submit description file should contain the submit command

```text
should_transfer_files = YES
```

With this command, all input and output files will be transferred as required to and from the scratch directory mounted as a Docker volume.

If no **executable** is specified in the submit description file, it is presumed that the Docker container has a default command to run.

When the job completes, is held, evicted, or is otherwise removed from the machine, the container will be removed.

## Submit File for Docker Job

Here is a complete submit description file for a sample docker universe job:

```text
universe                = docker
docker_image            = nvcr.io/nvidia/pytorch:20.06-py3
docker_network_type     = host
executable              = sub.sh
should_transfer_files   = YES
when_to_transfer_output = ON_EXIT

output                  = out.$(Cluster)
error                   = err.$(Cluster)
log                     = log.$(Cluster)

request_cpus            = 5
request_gpus            = 1
request_memory          = 3096
request_disk            = 10240

queue
```

A PyTorch container is the HTCondor job, and it runs the`sub.sh`script.

## Bash File for Docker Job

Here is a bash file to be executed for running`main.py`in a docker universe job:

```text
#!/bin/bash

cd /workspace/user_name/work_dir
python main.py
```

{% hint style="danger" %}
Note that it is mandatory to use`cd /workspace/user_name/work_dir`before your executable commmands \(`user_name`is your user name on DLS. `work_dir`is your local working directory that stores `main.py`under`/raid/home/user_name`on DLS\).
{% endhint %}





