# BOSH Errands

## Goal

BOSH Errands enable on demand execution of tasks within the deployment. This could be checking the status on a resource, adding a new user, running tests, or even `cf push`ing an application! Let's see an errand in action!

## Prerequisites

1. Obtain your workshop jumpbox public IP, Username, and Password - [Here]({{< relref "intro.md#workshop-credentials" >}})
1. Completed [lab 12]({{< relref "lab-12.md" >}})

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

## List possible Errands to run in the deployment:

1. Utilize the BOSH CLI to check the possible errands we could run on this deployment.

  - `bosh -e my-bosh -d zookeeper errands`

  - You should see something similar to the following:

            Using environment '10.0.0.6' as user 'admin' (openid, bosh.admin)

            Using deployment 'zookeeper'

            Name
            smoke-tests

            1 errands

            Succeeded

## Run a BOSH Errand:

1. While the BOSH CLI can tell us if the deployment is running successful it is important to have a smoke-test which can use a dedicated new instance to perform some sort of tests, probably upon the running cluster. Lets run it with the BOSH CLI.

  - `bosh -e my-bosh -d zookeeper run-errand smoke-tests`

  - You should see something similar to the following:

            Using environment '10.0.0.6' as user 'admin' (openid, bosh.admin)

            Using deployment 'zookeeper'

            Task 42

            Task 42 | 18:28:45 | Preparing deployment: Preparing deployment
            Task 42 | 18:28:45 | Warning: Ambiguous request: the requested errand name 'smoke-tests' matches both a job name and an errand instance group name. Executing errand on all relevant instances with job 'smoke-tests'.
            Task 42 | 18:28:45 | Preparing package compilation: Finding packages to compile (00:00:00)
            Task 42 | 18:28:45 | Preparing deployment: Preparing deployment (00:00:00)
            Task 42 | 18:28:45 | Creating missing vms: smoke-tests/40914c2a-081f-44b6-93d9-f618a0fa9e42 (0) (00:01:55)
            Task 42 | 18:30:40 | Updating instance smoke-tests: smoke-tests/40914c2a-081f-44b6-93d9-f618a0fa9e42 (0) (canary) (00:04:14)
            Task 42 | 18:34:54 | Running errand: smoke-tests/40914c2a-081f-44b6-93d9-f618a0fa9e42 (0) (00:00:08)
            Task 42 | 18:35:02 | Fetching logs for smoke-tests/40914c2a-081f-44b6-93d9-f618a0fa9e42 (0): Finding and packing log files (00:00:01)

## Examine the Output:

1. The log output shown from the errand should look similar to the following:

        Instance   smoke-tests/40914c2a-081f-44b6-93d9-f618a0fa9e42
        Exit Code  0
        Stdout     -----> simple test
                   Successfully created value
                   Successfully retrieved created value
                   Successfully set new value
                   Successfully deleted value
                   -----> phunt/zk-smoketests
                   Connecting to 10.0.0.12:2181
                   Connected in 31 ms, handle is 0
                   Connecting to 10.0.0.13:2181
                   Connected in 16 ms, handle is 1
                   Connecting to 10.0.0.14:2181
                   Connected in 16 ms, handle is 2
                   Connecting to 10.0.0.15:2181
                   Connected in 7 ms, handle is 3
                   Connecting to 10.0.0.16:2181
                   Connected in 31 ms, handle is 4
                   Connecting to 10.0.0.12:2181
                   Connected in 8 ms, handle is 5
                   Connecting to 10.0.0.16:2181
                   Connected in 31 ms, handle is 0
                   Smoke test successful
                   -----> phunt/zk-latencies (servers)
                   Connecting to 10.0.0.12:2181
                   Connected in 7 ms, handle is 0
                   Connecting to 10.0.0.13:2181
                   Connected in 7 ms, handle is 1
                   Connecting to 10.0.0.14:2181
                   Connected in 7 ms, handle is 2
                   Connecting to 10.0.0.15:2181
                   Connected in 7 ms, handle is 3
                   Connecting to 10.0.0.16:2181
                   Connected in 8 ms, handle is 4
                   Testing latencies on server 10.0.0.12:2181 using syncronous calls
                   created     100 permanent znodes  in    300 ms (3.007491 ms/op 332.503114/sec)
                   set         100           znodes  in    205 ms (2.059500 ms/op 485.554695/sec)
                   get         100           znodes  in     42 ms (0.424330 ms/op 2356.655073/sec)
                   deleted     100 permanent znodes  in    229 ms (2.297850 ms/op 435.189482/sec)
                   created     100 ephemeral znodes  in    230 ms (2.302339 ms/op 434.340888/sec)
                   watched     100           znodes  in     33 ms (0.332880 ms/op 3004.085375/sec)
                   deleted     100 ephemeral znodes  in    215 ms (2.151401 ms/op 464.813384/sec)
                   notif       100           watches in      0 ms (included in prior)
                   Testing latencies on server 10.0.0.13:2181 using syncronous calls
                   created     100 permanent znodes  in    196 ms (1.961172 ms/op 509.899280/sec)
                   set         100           znodes  in    170 ms (1.707389 ms/op 585.689490/sec)
                   get         100           znodes  in     42 ms (0.425601 ms/op 2349.618509/sec)
                   deleted     100 permanent znodes  in    176 ms (1.762681 ms/op 567.317623/sec)
                   created     100 ephemeral znodes  in    205 ms (2.059851 ms/op 485.472080/sec)
                   watched     100           znodes  in     37 ms (0.376520 ms/op 2655.900306/sec)
                   deleted     100 ephemeral znodes  in    182 ms (1.822979 ms/op 548.552536/sec)
                   notif       100           watches in      0 ms (included in prior)
                   Testing latencies on server 10.0.0.14:2181 using syncronous calls
                   created     100 permanent znodes  in    213 ms (2.131290 ms/op 469.199309/sec)
                   set         100           znodes  in    183 ms (1.834519 ms/op 545.102040/sec)
                   get         100           znodes  in     38 ms (0.388041 ms/op 2577.050308/sec)
                   deleted     100 permanent znodes  in    194 ms (1.945541 ms/op 513.995875/sec)
                   created     100 ephemeral znodes  in    200 ms (2.008798 ms/op 497.810103/sec)
                   watched     100           znodes  in     48 ms (0.486910 ms/op 2053.766189/sec)
                   deleted     100 ephemeral znodes  in    183 ms (1.833029 ms/op 545.545167/sec)
                   notif       100           watches in      0 ms (included in prior)
                   Testing latencies on server 10.0.0.15:2181 using syncronous calls
                   created     100 permanent znodes  in    218 ms (2.181051 ms/op 458.494598/sec)
                   set         100           znodes  in    191 ms (1.918571 ms/op 521.221264/sec)
                   get         100           znodes  in     42 ms (0.426419 ms/op 2345.112467/sec)
                   deleted     100 permanent znodes  in    206 ms (2.063830 ms/op 484.536056/sec)
                   created     100 ephemeral znodes  in    200 ms (2.004471 ms/op 498.884787/sec)
                   watched     100           znodes  in     37 ms (0.373690 ms/op 2676.013960/sec)
                   deleted     100 ephemeral znodes  in    262 ms (2.628431 ms/op 380.455062/sec)
                   notif       100           watches in      0 ms (included in prior)
                   Testing latencies on server 10.0.0.16:2181 using syncronous calls
                   created     100 permanent znodes  in    193 ms (1.937029 ms/op 516.254436/sec)
                   set         100           znodes  in    187 ms (1.871541 ms/op 534.319175/sec)
                   get         100           znodes  in     43 ms (0.437951 ms/op 2283.360009/sec)
                   deleted     100 permanent znodes  in    189 ms (1.895010 ms/op 527.701570/sec)
                   created     100 ephemeral znodes  in    185 ms (1.856110 ms/op 538.761145/sec)
                   watched     100           znodes  in     40 ms (0.400150 ms/op 2499.063962/sec)
                   deleted     100 ephemeral znodes  in    220 ms (2.200370 ms/op 454.469055/sec)
                   notif       100           watches in      0 ms (included in prior)
                   Latency test complete
                   -----> phunt/zk-latencies (cluster)
                   Connecting to 10.0.0.12:2181,10.0.0.13:2181,10.0.0.14:2181,10.0.0.15:2181,10.0.0.16:2181
                   Connected in 7 ms, handle is 0
                   Testing latencies on server 10.0.0.12:2181,10.0.0.13:2181,10.0.0.14:2181,10.0.0.15:2181,10.0.0.16:2181 using syncronous calls
                   created     100 permanent znodes  in    192 ms (1.929932 ms/op 518.153068/sec)
                   set         100           znodes  in    176 ms (1.766009 ms/op 566.248424/sec)
                   get         100           znodes  in     29 ms (0.299618 ms/op 3337.580469/sec)
                   deleted     100 permanent znodes  in    136 ms (1.367221 ms/op 731.410715/sec)
                   created     100 ephemeral znodes  in    186 ms (1.863248 ms/op 536.697108/sec)
                   watched     100           znodes  in     31 ms (0.318470 ms/op 3140.013176/sec)
                   deleted     100 ephemeral znodes  in    144 ms (1.448419 ms/op 690.407910/sec)
                   notif       100           watches in      0 ms (included in prior)
                   Latency test complete

        Stderr     2017/11/28 18:34:54 Connected to 10.0.0.15:2181
                   2017/11/28 18:34:54 Authenticated: id=315256189292118016, timeout=4000
                   2017/11/28 18:34:54 Re-submitting `0` credentials after reconnect
                   2017/11/28 18:34:54 Recv loop terminated: err=EOF
                   2017/11/28 18:34:54 Send loop terminated: err=<nil>


        1 errand(s)

1.  Notice that the  `stdout` and `stderr` is separated into 2 sections but the errand did succeed with `status 0`!
