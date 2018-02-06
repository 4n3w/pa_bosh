# Authoring a BOSH Release

## Goal

In [lab 7]({{< relref "lab-7.md" >}}) we deployed a pre-created BOSH release. How do we deploy a software system that is custom to our company? Let's explore this by building the same release deployed in [lab 7]({{< relref "lab-7.md" >}}) from the ground up.

## Prerequisites

1. Obtain your workshop jumpbox public IP, Username, and Password - [Here]({{< relref "intro.md#workshop-credentials" >}})
1. Completed [lab 7]({{< relref "lab-7.md" >}})

## Initialize the BOSH Release Structure:

1. Utilize your SSH Client to connect to your jumpbox.

  - `ssh username@jumpbox-ip`

1. Utilize the BOSH CLI to create the release, including a git repository

  - `bosh init-release --git --dir my-bosh-release`

1. Change directory into the newly created `my-bosh-release`

  - `cd my-bosh-release`

## Add Source Code to our BOSH Release:

1. Make a sub-directory `sample_app` under `src` this is where the application code will live. Copy the below bash script into a file called `app` and make it executable using `chmod`. Notice this script echos "My Custom APP STOUT!!" this is the only change from the earlier release we deployed in [lab 7]({{< relref "lab-7.md" >}}).

    `mkdir src/sample_app`

    `nano src/sample_app/app`

        #!/bin/bash
        while true
        do
            echo "My Custom APP STDOUT!!!"
            sleep 1
        done

    `chmod +x src/sample_app/app`

## Create a BOSH Package:

1. Create a BOSH Package which is responsible for compiling and moving the source code into a staging area.

  - `bosh generate-package sample_app`

1. Copy the below bash script into the `packaging` script located in the `packages/sample_app/` directory. This is what BOSH will execute to compile and move the results to the staging area.

    `nano packages/sample_app/packaging`

        set -e -x

        cp -a sample_app/* ${BOSH_INSTALL_TARGET}

1. Copy the below spec configuration into the `spec` file located in the `packages/sample_app/` directory. This spec file tells BOSH that the package depends on our source we added to the BOSH release earlier.

    `nano packages/sample_app/spec`

        ---
        name: sample_app

        dependencies: []

        files:
        - sample_app/**/*

## Create a BOSH Job:

1. Create a BOSH Job which is responsible for starting, stopping, and monitoring the application.

  - `bosh generate-job sample_job`

1. Copy the below bash script into a `ctl.erb` file under the `jobs/sample_job/templates/` directory. This script will be used to start, and stop the application.

    `nano jobs/sample_job/templates/ctl.erb`

        #!/bin/bash

        RUN_DIR=/var/vcap/sys/run/sample_job
        LOG_DIR=/var/vcap/sys/log/sample_job
        PIDFILE=${RUN_DIR}/pid

        case $1 in

          start)
            mkdir -p $RUN_DIR $LOG_DIR
            chown -R vcap:vcap $RUN_DIR $LOG_DIR

            echo $$ > $PIDFILE

            cd /var/vcap/packages/sample_app

              exec ./app \
              >>  $LOG_DIR/sample_app.stdout.log \
              2>> $LOG_DIR/sample_app.stderr.log

            ;;

          stop)
            kill -9 `cat $PIDFILE`
            rm -f $PIDFILE

            ;;

          *)
            echo "Usage: ctl {start|stop}" ;;

        esac

1. Copy the below monit configuration into the `monit` file located in the `jobs/sample_job/` directory. This file enables monit to monitor the running application by tracking its processid.

    `nano jobs/sample_job/monit`

        check process sample_app
          with pidfile /var/vcap/sys/run/sample_job/pid
          start program "/var/vcap/jobs/sample_job/bin/ctl start"
          stop program "/var/vcap/jobs/sample_job/bin/ctl stop"
          group vcap

1. Copy the below spec configuration into the `spec` file located in the `jobs/sample_job/` directory. This spec file tells BOSH that the job depends on our package we created earlier.

    `nano jobs/sample_job/spec`

        ---
        name: sample_job

        templates:
          ctl.erb: bin/ctl

        packages:
        - sample_app

        properties: {}

## Update the BOSH Release Metadata:

  1. Ensure the name of our release is `my-bosh-release` in the `final.yml` file located in the `config/` directory.

    `nano config/final.yml`

        name: my-bosh-release

## Create a BOSH Release Tarball:

  1. `bosh create-release --tarball=~/my-bosh-release.tgz --force`

    - You should see something similar to the following:

            Adding job 'sample_job/5765370e9d13c6c9d8a2db5631251e74fed718e9'...
            Adding package 'sample_app/31ac32410a7e59940847f585d72f9d0596e80132'...
            Added job 'sample_job/5765370e9d13c6c9d8a2db5631251e74fed718e9'
            Added package 'sample_app/31ac32410a7e59940847f585d72f9d0596e80132'

            Added dev release 'my-bosh/0+dev.1'

            Name         my-bosh
            Version      0+dev.1
            Commit Hash  empty+
            Archive      /home/bosh-1/my-bosh-release.tgz

            Job                                                  Digest                                    Packages
            sample_job/5765370e9d13c6c9d8a2db5631251e74fed718e9  871ec0f278ba78c1a1f9e357566bf912f2196687  sample_app

            1 jobs

            Package                                              Digest                                    Dependencies
            sample_app/31ac32410a7e59940847f585d72f9d0596e80132  4ceb58bbe0805126758886a77aca0fdc04c95edf  -

            1 packages

            Succeeded

## Upload our Custom BOSH Release

1. Upload the BOSH release tarball to the BOSH director.

  - `bosh -e my-bosh upload-release ~/my-bosh-release.tgz`

1. Ensure the release was uploaded and is ready for use!

  - `bosh -e my-bosh releases`

  - You should see something similar to the following:

            Using environment '10.0.0.6' as user 'admin' (openid, bosh.admin)

            Name                 Version  Commit Hash
            my-bosh              0+dev.1  empty+
            sample-bosh-release  0+dev.1  a2b19b8

            (*) Currently deployed
            (+) Uncommitted changes

            2 releases

            Succeeded

## Update our sample-bosh-deployment

1. To update our sample-bosh-deployment from [lab 7]({{< relref "lab-7.md" >}}), lets update our `sample-bosh-deployment` with our new custom release by editing the `sample-bosh-deployment.yml`

  - `bosh -e my-bosh -d sample-bosh-deployment manifest > ~/sample-bosh-deployment.yml`

  - `nano ~/sample-bosh-deployment.yml`

1. Change the release block to use our new release `my-bosh-release`.

1. Change the `sample_vm` instance in the `instance_groups` block to use the new release `my-bosh-release`

1. Ensure your `sample-bosh-deployment.yml` is the same as below and save/exit.

          name: sample-bosh-deployment

          releases:
          - {name: my-bosh-release, version: latest}

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
              release: my-bosh-release
            stemcell: trusty
            vm_type: default
          update:
            canaries: 1
            max_in_flight: 10
            canary_watch_time: 1000-30000
            update_watch_time: 1000-30000

1. Redeploy the instance with our new BOSH Release

  - `bosh -e my-bosh -d sample-bosh-deployment deploy ~/sample-bosh-deployment.yml`

1. When BOSH updates the software on our running BOSH deployment and instance it does not deploy a new instance instead it only updates the software that changed.

## View Instance Logs

1. To see the new log output change lets pull the logs from the updated instance using the BOSH CLI.

  - `cd ~`
  - `bosh -e my-bosh -d sample-bosh-deployment logs --job=sample_job`

1. Now we have the compressed logs lets extract them using `tar`.

  - `tar -xvf sample-bosh-deployment-20171128-172938-233692955.tgz`

1. `cat` the `sample_app.stdout.log` file in the `sample_job` directory.

1. Notice The job log output changed when we changed releases but did not delete old log output!
