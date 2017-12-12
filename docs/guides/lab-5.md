# Upload a BOSH Release

## Goal

Deploying software systems with BOSH is done with BOSH Releases. Lets upload a release to our BOSH director.

## Prerequisites

1. Obtain your workshop jumpbox public IP, Username, and Password - [Here]({{< relref "intro.md#workshop-credentials" >}})

## Upload the BOSH Release:

1. Utilize your SSH Client to connect to your jumpbox.

  - `ssh username@jumpbox-ip`

1. Download the [sample-bosh-release](https://github.com/Oskoss/bosh-release) using `wget`

  - `wget https://github.com/Oskoss/bosh-release/releases/download/v1.1/sample_release.tgz`

1. Upload the BOSH release tarball to the BOSH director.

  - `bosh -e my-bosh upload-release ~/sample_release.tgz`

1. Ensure the release was uploaded and is ready for use!

  - `bosh -e my-bosh releases`

  - You should see something similar to the following:

            Using environment '10.0.0.6' as user 'admin' (openid, bosh.admin)

            Name                 Version  Commit Hash
            sample-bosh-release  0+dev.1  a2b19b8

            (*) Currently deployed
            (+) Uncommitted changes

            1 releases

            Succeeded
