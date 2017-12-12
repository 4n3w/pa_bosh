# Create and Upload a BOSH Cloud-Config

## Goal

In order for BOSH to communicate with specific infrastructures (GCP/AWS/vSphere/etc.) we abstract these details into a BOSH cloud-config. Lets create and upload our own to the BOSH director.

## Prerequisites

1. Obtain your workshop jumpbox public IP, Username, and Password - [Here]({{< relref "intro.md#workshop-credentials" >}})
1. Obtain your GCP Network Name - [Here]({{< relref "intro.md#workshop-credentials" >}})

## Part 1: Creating the Cloud Config

1. Utilize your SSH Client to connect to your jumpbox.

  - `ssh username@jumpbox-ip`

1. Utilize your favorite command-line text editor to create a `cloud-config.yml` text file. (We will use `nano`, but `vi` works as well!)

  - `nano cloud-config.yml`

1. Paste the following into the `cloud-config.yml`.

        azs:
        - name: z1
          cloud_properties: {zone: us-central1-a}
        - name: z2
          cloud_properties: {zone: us-central1-f}

        vm_types:
        - name: default
          cloud_properties:
            machine_type: n1-standard-1
            root_disk_size_gb: 20
            root_disk_type: pd-ssd
        - name: large
          cloud_properties:
            machine_type: n1-standard-2
            root_disk_size_gb: 20
            root_disk_type: pd-ssd
        disk_types:
        - name: default
          disk_size: 3000

        networks:
        - name: default
          type: manual
          subnets:
          - range:   10.0.0.0/16
            gateway: 10.0.0.1
            dns:     [8.8.8.8, 8.8.4.4]
            reserved: [10.0.0.0-10.0.0.10]
            azs:     [z1, z2]
            cloud_properties:
              ephemeral_external_ip: true
              subnetwork_name: <YOUR_GCP-SUBNET-NAME>
              network_name: <YOUR-GCP-NETWORK-NAME>
              ephemeral_external_ip: true
              tags: [bosh]
        - name: vip
          type: vip

        compilation:
          workers: 3
          reuse_compilation_vms: true
          az: z1
          vm_type: default
          network: default

1. Change the value of the `subnetwork_name: <YOUR_GCP-SUBNET-NAME>` and `network_name: <YOUR-GCP-NETWORK-NAME>` tags to your GCP subnet and GCP network name provided earlier [here]({{< relref "intro.md#workshop-credentials" >}}).

## Part 2: Understanding the Cloud Config

When deploying services we have to define the following:

  1. `azs` or availability zones allow us to provide high availability to our services deployed with BOSH. Here we create 2 zones, z1, and z2.

  1. `vm_types` allow us to t-shirt size the vms that make up our services deployed with BOSH. Here we create a default vm_type that contains a Standard machine with 4 virtual CPUs, 15 GB of memory, and a 20GB SSD disk.

  1. `disk_types` describe what type disks can be attached as a persistent disk to the vms that make up our services deployed with BOSH. In this case we create a 3GB disk.

  1. `networks` define what ips, gateways, dns, and specific cloud provider networking properties are needed. Here we will use the `10.0.0.0 - 10.0.255.255` subnet with google dns `8.8.8.8`, and the GCP network that you replaced.

  1. `compilation` specifies which of all the above resources BOSH should use when compiling code.

## Part 3: Uploading the Cloud Config to the BOSH director

1. Upload the `cloud-config.yml` to the BOSH director. All services we deploy with BOSH will use this from now on.

  - `bosh -e my-bosh update-cloud-config cloud-config.yml`
