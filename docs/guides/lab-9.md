# BOSH Instance Internals

## Goal

Persistence and monitoring are two of the most important pieces of a BOSH release. Lets explore how both of these are done, and where the important files are by examining our deployed sample-bosh-release.

## Prerequisites

1. Obtain your workshop jumpbox public IP, Username, and Password - [Here]({{< relref "intro.md#workshop-credentials" >}})
1. Completed [lab 7]({{< relref "lab-7.md" >}})

## Part 1: How does persistence work with BOSH?

1. Once `SSH`ed into the BOSH instance lets understand how and where BOSH places persistent disks vs emphimeral disks.

  - `df -h`

  - You should see something like the following:

            Filesystem      Size  Used Avail Use% Mounted on
            udev            7.4G  4.0K  7.4G   1% /dev
            tmpfs           1.5G  268K  1.5G   1% /run
            /dev/sda1       2.8G  1.2G  1.5G  46% /
            none            4.0K     0  4.0K   0% /sys/fs/cgroup
            none            5.0M     0  5.0M   0% /run/lock
            none            7.4G     0  7.4G   0% /run/shm
            none            100M     0  100M   0% /run/user
            /dev/sda3       8.4G   22M  7.9G   1% /var/vcap/data
            tmpfs           1.0M  4.0K 1020K   1% /var/vcap/data/sys/run
            /dev/sdb1       4.8G   10M  4.6G   1% /var/vcap/store

  - Notice that the persistent disk we specified in our `sample-bosh-manifest.yml` (shown below for reference) is attached to the `/var/vcap/store` mount point. This requires that any software system deployed with BOSH write any persistent data to this directory or a subdirectory.

  - **cat sample-bosh-manifest.yml**


            name: sample-bosh-deployment

            releases:
            - {name: sample-bosh-release, version: latest}

            stemcells:
            - alias: trusty
              os: ubuntu-trusty
              version: latest

            instance_groups:
            - name: sample_vm
              instances: 1
              networks:
              - name: default
              azs: [z1]
              jobs:
              - name: sample_job
                release: sample-bosh-release
              stemcell: trusty
              vm_type: default
              persistent_disk: 5120
            update:
              canaries: 1
              max_in_flight: 10
              canary_watch_time: 1000-30000
              update_watch_time: 1000-30000

  - This allows us the ability to stop, terminate, or delete the BOSH instance, with our persistent disk data remaining intact and also is how BOSH is able to update BOSH instances in the future without losing data.

## Part 2: How does BOSH monitor software packages?

1. Once `SSH`ed into the BOSH instance lets understand how BOSH monitors running BOSH jobs in a BOSH release.

    - `sudo su`
    - `monit summary`

    - You should see something like the following:

            The Monit daemon 5.2.5 uptime: 19m

            Process 'sample_app'                running
            System 'system_localhost'           running

    - For each BOSH job we are running on the BOSH instance we will see a `Process` type entry. Here we have only one since that is all we requested back in our `sample-bosh-manifest.yml`.

    - Also notice that there is a `System` type entry. This is the entry for the overall system including the BOSH agent itself.

1. When we want to restart a job all we need to do is ask monit to do it for us.

    - `monit restart sample_app`

    - The same command is used for `restart`, `start`, and `stop`.

## Part 3: How does a BOSH Release get deployed on the running BOSH instance?

1. When deploying BOSH releases, BOSH places compiled code and other resources in the `/var/vcap/` directory tree, which BOSH creates on the BOSH instances. Two directories seen in the BOSH release, `jobs`, and `packages` appear on BOSH instances as `/var/vcap/jobs` and `/var/vcap/packages` respectively. Once `SSH`ed into the BOSH instance lets use the `tree` utility to show these.

- `sudo apt-get install tree`
- `tree /var/vcap`

## Part 4: Where can we see logs?

1. All logs on the BOSH instances are stored relative to which BOSH job they come from starting from the `/var/vcap/sys/log/` directory. Once `SSH`ed into the BOSH instance lets use the `tree` utility to show these.

  - `sudo apt-get install tree`
  - `tree /var/vcap/sys/log/`

1. Exit the vm when done.

    - `exit`
    - `exit`
