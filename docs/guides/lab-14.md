# BOSH Release Upgrades

## Goal

BOSH makes day 2 operations easy! Lets upgrade our zookeeper cluster to the latest version.

## Prerequisites

1. Obtain your workshop jumpbox public IP, Username, and Password - [Here]({{< relref "intro.md#workshop-credentials" >}})
1. Completed [lab 13]({{< relref "lab-13.md" >}})

## Ensure our zookeeper is deployed:

1. Utilize your SSH Client to connect to your jumpbox.

  - `ssh username@jumpbox-ip`

1. Utilize the BOSH CLI to check the status of the deployment.

  - `bosh -e my-bosh -d zookeeper instances -p`

  - You should see something similar to the following:

            Using environment '10.0.0.6' as user 'admin' (openid, bosh.admin)

            Task 41. Done

            Deployment 'zookeeper'

            Instance                                          Process    Process State  AZ  IPs
            smoke-tests/40914c2a-081f-44b6-93d9-f618a0fa9e42  -          -              z1  -
            zookeeper/185f9261-e51f-46b5-87f4-38e3e56f61a9    -          running        z1  10.0.0.12
            ~                                                 zookeeper  running        -   -
            zookeeper/35166c8e-0fc4-43cd-b957-b612af9fcf9d    -          running        z2  10.0.0.15
            ~                                                 zookeeper  running        -   -
            zookeeper/5bdf8efd-f80a-4ab1-b0ad-31f7271e56cc    -          running        z1  10.0.0.13
            ~                                                 zookeeper  running        -   -
            zookeeper/618abfbb-cf6e-4828-bd9e-3bbd85b8acf0    -          running        z2  10.0.0.16
            ~                                                 zookeeper  running        -   -
            zookeeper/d3eaa357-0920-4f1e-a63c-eb1adaf3243c    -          running        z1  10.0.0.14
            ~                                                 zookeeper  running        -   -

            11 instances

            Succeeded

## Upload the latest zookeeper BOSH Release

1. Utilize the BOSH CLI to upload the latest zookeeper release.

  - `bosh -e my-bosh upload-release https://bosh.io/d/github.com/cppforlife/zookeeper-release?v=0.0.7 --sha1 a863ef216ea75d22950703a3c924959cb20e59ba`

1. Ensure the release was uploaded and is ready for use!

  - `bosh -e my-bosh releases`

  - You should see something similar to the following:

            Using environment '10.0.0.6' as user 'admin' (openid, bosh.admin)

            Name                 Version   Commit Hash
            my-bosh              0+dev.1   empty+
            sample-bosh-release  0+dev.1   a2b19b8
            zookeeper            0.0.7     e7ead22
            ~                    0.0.6     2f8926f

            (*) Currently deployed
            (+) Uncommitted changes

            2 releases

            Succeeded

## Update our zookeeper deployment

1. To update our zookeeper deployment from [lab 12]({{< relref "lab-12.md" >}}) utilize the BOSH CLI to Redeploy.

  - `bosh -e my-bosh -d zookeeper deploy ~/zookeeper-manifest.yml`

  - Notice that we did not change any values in the `zookeeper-manifest.yml`! This is because we specified `latest` under the releases block in the `zookeeper-manifest.yml`.
