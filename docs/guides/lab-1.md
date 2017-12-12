#Setting Up the BOSH CLI

## Goal

When creating a BOSH director there are many credentials that must be created. These credentials are generated and stored in the creds.yml file.
In order for us to connect to the director we will utilize both Username, Password and a Certificate.

## Prerequisites

1. Obtain your workshop jumpbox public IP, Username, and Password - [Here]({{< relref "intro.md#workshop-credentials" >}})

## Part 1: Alias your BOSH Director

1. Utilize your SSH Client to connect to your jumpbox.

  - `ssh username@jumpbox-ip`

1. View the generated credentials within the `creds.yml` file

  - `cat creds.yml`

1. Rather than specifying the director IP and certificate for each BOSH command we run, lets alias those into a reference name. This will allow subsequent BOSH CLI commands to only require the reference name via the --environment flag (-e)

  - `bosh alias-env my-bosh -e 10.0.0.6 --ca-cert <(bosh int ./creds.yml --path /director_ssl/ca)`

## Part 2: Login to your BOSH Director

1. To find the correct password lets use the BOSH CLI to parse the `creds.yml` for us. Note this down for the next step.

  - `bosh int ./creds.yml --path /admin_password`

2. Login to the director with `admin` as the username and the password from the last step.

  - `bosh -e my-bosh login`
