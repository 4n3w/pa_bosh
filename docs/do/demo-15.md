# BOSH Runtime-Config & Add-ons

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

## Upload the OS-Conf BOSH Release

1. Utilize the BOSH CLI to upload the latest os-conf release that we can use to add an SSH banner to our BOSH zookeeper instances.

  - `bosh -e my-bosh upload-release https://bosh.io/d/github.com/cloudfoundry/os-conf-release?v=19 --sha1 f515406949ee0bba0329d1ce4a7eb1679521eabd`

## Create a BOSH Runtime Config

1. BOSH has a way to specify global configuration for all VMs in all deployments, it is called the Runtime-Config.
1. To update our zookeeper deployment from [lab 12]({{< relref "lab-12.md" >}}) by adding an SSH banner, we first have to create a runtime-config.

  - `nano runtime-config.yml`

1. Copy the following into the `runtime-config.yml` file we just created.

        releases:
        - name: os-conf
          version: 19

        addons:
        - name: misc
          jobs:
          - name: login_banner
            release: os-conf
          properties:
            login_banner:
              text: |
                Welcome to Pivotals Workshop!
                BBBB   OOO   SSS  H  H    III  OOO
                B   B O   O S     H  H     I  O   O
                BBBB  O   O  SSS  HHHH     I  O   O
                B   B O   O     S H  H ..  I  O   O
                BBBB   OOO  SSSS  H  H .. III  OOO
                This computer system is for authorized use only. All activity is logged and
                regularly checked by system administrators. Individuals attempting to connect to,
                port-scan, deface, hack, or otherwise interfere with any services on this system
                will be reported.

## Upload the BOSH Runtime Config

1. Now lets use the BOSH CLI to upload the `runtime-config.yml`.

  - `bosh -e my-bosh update-runtime-config runtime-config.yml`

## Apply the BOSH Runtime Config

1. BOSH only applies runtime-config changes to the deployment during a `bosh deploy`. We must redeploy the zookeeper deployment as a last step to get our SSH banner.

  - `bosh -e my-bosh -d zookeeper deploy ~/zookeeper-manifest.yml`

## Test the SSH Banner

1. To test the SSH Banner lets SSH to a BOSH instance. Review [lab 8]({{< relref "lab-8.md" >}}) for understanding access to BOSH instances.

  - `bosh -e my-bosh -d zookeeper instances`
  - `bosh -e my-bosh -d zookeeper ssh zookeeper/185f9261-e51f-46b5-87f4-38e3e56f61a9`

  - You should see something similar to the following:

            Using environment '10.0.0.6' as user 'admin' (openid, bosh.admin)

            Using deployment 'zookeeper'

            Task 51. Done
            Welcome to Pivotals Workshop!
            BBBB   OOO   SSS  H  H    III  OOO
            B   B O   O S     H  H     I  O   O
            BBBB  O   O  SSS  HHHH     I  O   O
            B   B O   O     S H  H ..  I  O   O
            BBBB   OOO  SSSS  H  H .. III  OOO
            This computer system is for authorized use only. All activity is logged and
            regularly checked by system administrators. Individuals attempting to connect to,
            port-scan, deface, hack, or otherwise interfere with any services on this system
            will be reported.

            Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 4.4.0-97-generic x86_64)
