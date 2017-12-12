# Accessing a BOSH Managed Instance

## Goal

BOSH runs software systems on virtual machines referred to as BOSH instances. Since BOSH deploys instances behind firewalls, without public IPs, and with secured ssh keys, it can be hard to get access to these machines via the typical SSH client. Lets explore how to access these machines via the BOSH CLI directly.

## Prerequisites

1. Obtain your workshop jumpbox public IP, Username, and Password - [Here]({{< relref "intro.md#workshop-credentials" >}})
1. Completed [lab 7]({{< relref "lab-7.md" >}})

## Part 1: Accessing a deployed BOSH instance

1. Utilize your SSH Client to connect to your jumpbox.

  - `ssh username@jumpbox-ip`

1. Ensure you have run through [lab 6]({{< relref "lab-6.md" >}}) and have a sample-bosh-deployment running.

  - `bosh -e my-bosh deployments`

  - You should see something like the following:

            Using environment '10.0.0.6' as user 'admin' (openid, bosh.admin)

            Name                    Release(s)    Stemcell(s)                                    Team(s)  Cloud Config
            sample-bosh-deployment  bosh/0+dev.1  bosh-google-kvm-ubuntu-trusty-go_agent/3468.5  -        latest

            1 deployments

            Succeeded

1. Find the instance we want to SSH into by listing all instances in our deployment.

    - `bosh -e my-bosh -d sample-bosh-deployment instances`

    - You should see something like the following, note down the `instance_name/uuid`:

            Using environment '10.0.0.6' as user 'admin' (openid, bosh.admin)

            Task 82. Done

            Deployment 'sample-bosh-deployment'

            Instance                                        Process State  AZ  IPs
            sample_vm/71160b1a-faaa-487c-b107-7d4ed8fce7ce  running        z1  10.0.0.11

            1 instances

            Succeeded

1. Using the `instance_name/uuid` utilize the BOSH CLI to access the instance via the `ssh` subcommand.

    - `bosh -e my-bosh -d sample-bosh-deployment ssh sample_vm/71160b1a-faaa-487c-b107-7d4ed8fce7ce`

    - If successful you should now be inside the instance and have full `root` access via `sudo su`.
