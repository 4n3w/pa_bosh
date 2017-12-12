# Reconfiguring BOSH Deployments

## Goal

There are many tools that attempt to concur automating infrastructure, but BOSH was designed not only with this in mind but also what we call "day 2 operations". Lets take a quick look at how easy it is to expand a BOSH instance.

## Prerequisites

1. Obtain your workshop jumpbox public IP, Username, and Password - [Here]({{< relref "intro.md#workshop-credentials" >}})
1. Completed [lab 7]({{< relref "lab-7.md" >}})

## Expand the BOSH Instance Size:

1. Utilize your SSH Client to connect to your jumpbox.

  - `ssh username@jumpbox-ip`

1. Utilize the BOSH CLI to show how much

  - `bosh -e my-bosh -d sample-bosh-deployment instances --vitals`

1. SSH into the BOSH instance in our `sample-bosh-deployment`.

  - `bosh -e my-bosh -d sample-bosh-deployment ssh sample_vm/3f361ef3-7a9d-4e7d-a167-9bf5ad033c02`

1. Check the number of CPU cores via the `nproc` utility. Note this down.

  - `nproc --all`

1. Check the memory capacity via the `free` utility. Note this down.

  - `free -g`

1. Exit the BOSH instance.

  - `exit`

1. Edit the `sample-bosh-manifest.yml` text file on your jumpbox. (We will use `nano`, but `vi` works as well!)

  - `nano sample-bosh-manifest.yml`

1. Change the `vm_type: default` line to `vm_type: large`. This will upgrade the t-shirt sizing for the BOSH instance.

1. Save the file.

1. Have BOSH apply the changes to the deployment.

  - `bosh -e my-bosh -d sample-bosh-deployment deploy sample-bosh-manifest.yml`

  - Notice BOSH always will show the difference between what is currently deployed and what you are asking to be changed:

            Using environment '10.0.0.6' as user 'admin' (openid, bosh.admin)

            Using deployment 'sample-bosh-deployment'

              instance_groups:
              - name: sample_vm
            -   vm_type: default
            +   vm_type: large

1. SSH back into the BOSH instance in our `sample-bosh-deployment`.

1. Check the number of CPU cores via the `nproc` utility. This should be larger than before!

  - `nproc --all`

1. Check the memory capacity via the `free` utility. This should be larger than before!

  - `free -g`

1. Exit the BOSH instance.

  - `exit`

1. Once back on the jumpbox use the BOSH CLI to show the overall vitals for the BOSH instance. This is handy when determining if instances are overloaded.

  - `bosh -e my-bosh -d sample-bosh-deployment instances --vitals`

              Using environment '10.0.0.6' as user 'admin' (openid, bosh.admin)

              Task 106. Done

              Deployment 'sample-bosh-deployment'

              Instance                                        Process State  AZ  IPs        VM Created At                 Uptime         Load              CPU    CPU   CPU   CPU   Memory       Swap      System      Ephemeral   Persistent
                                                                                                                                         (1m, 5m, 15m)     Total  User  Sys   Wait  Usage        Usage     Disk Usage  Disk Usage  Disk Usage
              sample_vm/3f361ef3-7a9d-4e7d-a167-9bf5ad033c02  running        z1  10.0.0.11  Tue Nov 28 04:57:40 UTC 2017  0d 0h 15m 39s  0.00, 0.00, 0.00  -      0.0%  0.1%  0.1%  1% (152 MB)  0% (0 B)  43% (33i%)  0% (0i%)    0% (0i%)

              1 instances

              Succeeded
