# Configuring RollingUpdate Deployments

In this lab you will learn how to configure your Deployment to perform
rolling updates without downtime.

# Learning Outcomes

After completing the lab, you will be able to:

- Describe the value of rolling updates
- Explain how readiness probes make rolling updates better
- Describe how to configure readiness probes for a pod in Kubernetes
- Paraphrase the reason why a preStop hook is necessary

## Deployment Strategies

By default, the deployment strategy used by Kubernetes is
`RollingUpdate`.
During a deployment, the `RollingUpdate` strategy will start new Pod(s)
to run your application before terminating the old Pod(s).

## Deploying without a readiness probe

It might not have been obvious when deploying the update from `v0` to
`v1` of your image, but your app actually experienced downtime despite
using the `RollingUpdate` strategy.
You will start by verifying that there is in fact downtime when you do
deploys.

Ensure you execute the following commands within 5 minutes from start,
to finish.
The timing is important,
as you want to simulate load during the deployment.

1.  Edit your Deployment to go back to the version tagged `v0`, but
    do not apply the Deployment yet.

1.  Start watching your Pods using the `kubectl get pods --watch`
    command.

1.  Generate some load using the `run-load-test` script.
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

    Feel free to tune up or down the duration of the test as necessary
    for you to complete the following series of steps.

1.  In a third terminal window apply the Deployment while the load
    test is still running.

    Watch the output of the load test.
    You will see the output from `v1` initially, followed by failures,
    followed by the output from `v0`.

1.  Inspect the output of the `kubectl get pods --watch`.
    You will see the new Pod is created and running before the old Pod
    is terminated.
    Even though the new Pod and its containers were running, the
    application itself was not ready to handle requests.
    The same thing will happen when horizontally scaling the app:
    traffic will be routed to the new Pods before they are ready to
    handle requests.
    Configuring a readiness probe will fix both of these problems.

1.  Verify whether or not your load test is complete by reviewing the
    output of your load test.
    Issue a SIGQUIT (Ctrl+C) to the terminal window where you ran the
    `run-load-test` command.

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

## Configuring a readiness probe

Actuator provides an endpoint specifically used to determine if the
application is ready to handle traffic.
You will configure Kubernetes to use this Actuator endpoint.

1.  Build a new image with version `v2`.
    This new image will contain your latest code including Actuator.

1.  Push your new image to Docker Hub.

1.  Modify the Deployment to configure a readiness probe:

    ```diff
        spec:
          containers:
            - name: pal-tracker-container
              image: YOUR_DOCKER_HUB_USERNAME/pal-tracker:v0
    +         readinessProbe:
    +           httpGet:
    +             path: /actuator/health/readiness
    +             port: 8080
    ```

    This readiness probe will perform an HTTP GET to the
    `/actuator/health/readiness` path of the application on port 8080.
    If the application returns a status code greater than or equal to
    200 and less than 400, the probe is considered a success meaning the
    Pod and its application is alive, healthy and ready to receive
    traffic.

1.  Update the container version to `v2` in the Deployment YAML.

1.  Run another load test.

1.  Apply the Deployment.

1.  Inspect the output of the load test.
    You should see less errors during this deploy, however it is still
    possible you will see some errors.

    This is due to the distributed and asynchronous nature of
    Kubernetes.
    Anytime a Pod is terminated, the removal of the routing rule for
    traffic to the Pod happens asynchronously with the terminating of
    the Pod itself.
    This eventual consistency means that it is possible for traffic to
    get routed to a terminated Pod.

1.  Terminate the load test or let it complete before you proceed with
    the next section.

## Configure a preStop lifecycle hook

At this point in time, the best solution is to wait an indeterminate
amount of time before terminating the Pod.
This does not guarantee that traffic will not be routed to the Pod after
its terminated,
but it makes it much less likely.

1.  Update your Deployment to include a `preStop` lifecycle hook:

    ```diff
              readinessProbe:
                httpGet:
                  path: /
                  port: 8080
    +         lifecycle:
    +           preStop:
    +             exec:
    +               command: [ "/bin/sleep", "10" ]
    ```

    This lifecycle hook will run `/bin/sleep 10` before a TERM signal is
    sent to the Java process running inside the container.
    The result is the Pod will wait an additional 10 seconds before
    starting to terminate.
    Empirical testing has shown that it can take up to 10 seconds to be
    updated and stop routing traffic to the Pod, although the time
    may vary depending on your deployment topology or other factors.

    You need to have the running Pod that is about to be terminated to
    have the `preStop` hook configured before testing.

1.  Change the value of your `welcome.message` in your ConfigMap yaml.
    This will allow you to see when your application has successfully
    deployed the latest version while running your load test.

1.  Run a load test again.

1.  Apply the changes to both your ConfigMap and your Deployment by
    running:

    ```bash
    kubectl apply -f k8s/
    ```

    This time you will see no errors when performing the deploy.

1.  Either let the load test complete,
    or terminate it before wrapping up the lab.

## Wrap up

1. Commit and push your changes.

# Assignment

Submit the assignment using the `cloudNativeDeveloperK8sRollingUpdate`
Gradle task.
It requires you to provide the name of your Kubernetes Deployment.

For example:

```bash
cd ~/workspace/assignment-submission
./gradlew cloudNativeDeveloperK8sRollingUpdate -PdeploymentName=pal-tracker
```

# Learning Outcomes

Now that you have completed the lab, you should be able to:
::learningOutcomes::

# Resources

- [Liveness and Readiness Probes Overview](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
- [Probe API Documentation](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#probe-v1-core)
- [Termination of Pods](https://kubernetes.io/docs/concepts/workloads/pods/#termination-of-pods)
- [Delaying Shutdown to Wait for Pod Deletion Propagation](https://blog.gruntwork.io/delaying-shutdown-to-wait-for-pod-deletion-propagation-445f779a8304)