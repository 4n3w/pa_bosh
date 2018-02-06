# Upload a BOSH Stemcell

## Goal

In order to deploy services with BOSH we need a base operating system -- a BOSH Stemcell. Lets upload one to the BOSH director.

### Prerequisites

1. Obtain your workshop jumpbox public IP, Username, and Password - [Here]({{< relref "intro.md#workshop-credentials" >}})

## Upload the Stemcell

1. Utilize your SSH Client to connect to your jumpbox.

  - `ssh username@jumpbox-ip`

1. Ensure your BOSH environment is still in a good state

  - `bosh -e my-bosh environment`

1. Download a fresh Google Cloud Platform Stemcell onto your jumpbox [here](https://s3.amazonaws.com/bosh-core-stemcells/google/bosh-stemcell-3468.5-google-kvm-ubuntu-trusty-go_agent.tgz).

  - `wget --content-disposition https://s3.amazonaws.com/bosh-core-stemcells/google/bosh-stemcell-3468.5-google-kvm-ubuntu-trusty-go_agent.tgz`

1. Upload the Stemcell to the BOSH director

  - `bosh -e my-bosh upload-stemcell bosh-stemcell-*-google-kvm-ubuntu-trusty-go_agent.tgz`

1. Check to ensure the upload was successful

  - `bosh -e my-bosh stemcells`
