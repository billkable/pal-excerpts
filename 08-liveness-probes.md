# Liveness Probes

This lab will walk you through how to setup a liveness probe to monitor
the health of your Pods.

# Learning Outcomes

After completing the lab, you will be able to:

- Describe how to setup a Kubernetes liveness probe to monitor Pod health

# Configure a liveness probe

Liveness probes are a good way to provide application specific insight
into the health of your application to Kubernetes.
By configuring the liveness probe, Kubernetes will be able to restart
and scale Pods according to the status of liveness rules you may have
created in your applications.

1.  Edit the `k8s/deployment.yaml` file:

    ```diff
              readinessProbe:
                httpGet:
                  path: /actuator/health/readiness
                  port: 8080
    +         livenessProbe:
    +           httpGet:
    +             path: /actuator/health/liveness
    +             port: 8080
    +           initialDelaySeconds: 150
    ```

    This liveness probe will perform an HTTP GET to the
    `/actuator/health/liveness` endpoint on port 8080.
    If the application returns a status code greater than or equal to
    200 and less than 400, the probe is considered a success meaning the
    Pod and its application is alive and healthy.
    The `initialDelaySeconds` is the number of seconds to wait before
    running the liveness probe after a container is started.
    150 seconds will give your application plenty of time to boot before
    Kubernetes starts checking it for liveness.

1.  Add an environment variable named `MANAGEMENT_ENDPOINTS_WEB_EXPOSURE_INCLUDE`
    with the value of `info,health,palTrackerFailure`.
    You will need to change both the `k8s/deployment.yaml` and the
    `k8s/configmap.yaml` files.

    This will expose the `/actuator/palTrackerFailure` endpoint for use
    in future steps.

    **Take care when exposing mangement endpoints in production
    environments because they can include sensitive information.**

1.  Apply your changes and wait for deploy to finish.

# Demonstrate the liveness probe

1.  Before exercising the liveness probe, watch the Pods by running:

    ```bash
    kubectl get pods --watch
    ```

1.  In a separate terminal window,
    tail the logs of the pod:

    ```bash
    kubectl logs pod/<pod name> -f
    ```

1.  In another terminal window,
    generate some load using the `run-load-test` script.
    If you have been supplied a remote desktop by your instructor,
    it will be provided for you in the path.
    Otherwise you can
    [download it and put it in your path](https://raw.githubusercontent.com/platform-acceleration-lab/docker-loadtest/main/run-load-test).
    You can checks its usage via `run-load-test -h`.

    Configure and run it with the following parameters:

    -   Duration:
        `300` seconds (5 minutes)

    -   Concurrent users:
        `10`

    -   Workrate (requests per second):
        `10`

    -   Kubernetes namespace:
        `development`

    -   URL:
        `http://<base domain for your K8s installation>`

    Notice the output of the load test is running in the terminal window.
    The test is actually running in the background on Kubernetes,
    so you can terminate the terminal window and the test will continue
    to run.

    [Read here about how the load test generator works](https://github.com/platform-acceleration-lab/docker-loadtest#running-with-the-wrapper-scripts-in-k8s).

1.  In a different terminal window run the following curl command to
    simulate a failure:

    ```bash
    curl -v -XPOST http://YOUR_APPLICATION_URL/actuator/palTrackerFailure
    ```

1.  View the output of the Pod watch command for up to 60 seconds,
    as well as the pod log output.

    -   Kubernetes will detect the failure of the liveness probe based
        on the default liveness probe `periodSeconds` configuration of
        10 seconds.

    -   Kubernetes will retry on failed probes based on the default
        liveness probe `failureThresholds` configuration of 3 times.

    -   After the factor of the probe `periodSeconds` and
        `failureThresholds`,
        Kubernetes will give up on the pod,
        dispose of it,
        and restart it.

        See more about the
        [Kubernetes Probe configuration](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#configure-probes).

    -   You will see the shutdown occur in the logs,
        and the log tail will terminate
        (because the pod has been disposed).

    -   In the Pod watch you will see the `READY` value go from `1/1` to
        `0/1`.
        This means that the one container running within the Pod is has
        been recreated,
        but not yet ready.
        You will also see the `RESTARTS` count go up.

    -   You will see the `READY` value go from `0/1` to `1/1`.
        This means your pod has been successfully started,
        and is ready for work.

    -   Watch the output of the load test.
        Do you see errors?
        Why?

    -   Run the following command to view the pod description:

        ```bash
        kubectl describe pod/<pod name>
        ```

        View the `Events` section.
        You will see the correlated event of liveness probe failures,
        pod disposal,
        and creation.

1.  If your load test has not yet completed,
    terminate it by running the `abort-load-test` command.

    If you have been supplied a remote desktop by your instructor,
    it will be provided for you in the path.
    Otherwise you can
    [download it and put it in your path](https://raw.githubusercontent.com/platform-acceleration-lab/docker-loadtest/main/abort-load-test).
    You can checks its usage via `abort-load-test -h`.

    Configure and run it with the following parameters:

    -   Kubernetes namespace:
        `development`

    This will terminate the load test,
    and delete the `loadtest` pod running  in the K8s cluster.

# Availability

Note that if you only had one Pod running before and during the failure
simulation,
then you will find requests to your application would return a `503`
status between the time Kubernetes disposed of your unhealthy Pod,
and its recovery.

If you had more than one Pod running before simulating the failure,
then you will not experience any downtime for your application.
Kubernetes will stop routing traffic to the Pod that is currently
unhealthy and only the healthy Pods will handle traffic.

Kubernetes supports the ability to handle this scenario by adding a
[ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/).

The purpose of a ReplicaSet is to **maintain a stable set of Pods**
**running our application to help guarantee availability**.

***Important Note:***
***ReplicaSet is a Kubernetes feature that may be used to help achieve***
***higher levels of availability, or horizontal scaling.***
***This lesson focuses on the concept of Availability.***
***The concept of Scalability is covered at more depth in another***
***lesson.***

You will configure the existing ReplicaSet in your deployment to address
the availability concern.

1.  Apply a replica factor of 2 to your application:

    ```bash
    kubectl scale deploy/pal-tracker --replicas=2
    ```

1.  Repeat steps 1, 3 through 6 of
    [Demonstrate the liveness probe](#demonstrate-the-liveness-probe)
    section (eliminating watching the logs).

1.  When observing the load test output following the disposal and
    re-creation of the failed pod,
    do you see errors?
    Why not?

1.  Revert the replicas to 1:

    ```bash
    kubectl scale deploy/pal-tracker --replicas=1
    ```

1.  Notice we solve the issue of availability during the transient
    failure by adding redundant pods through the `kubectl scale`
    command.

    Notice we are not talking about scaling in the lab,
    you will work with scaling further in another lab.

1.  If you have not already, commit and push your changes to GitHub.

# Submit this assignment

Submit the assignment using the `cloudNativeDeveloperK8sLivenessProbe`
gradle task from within the existing `assignment-submission` project
directory.
It requires you to provide the name of your Deployment in your review
namespace.

For example:

```bash
cd ~/workspace/assignment-submission
./gradlew cloudNativeDeveloperK8sLivenessProbe -PdeploymentName=pal-tracker
```

# Resources

- [Probe API Documentation](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#probe-v1-core)
- [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)