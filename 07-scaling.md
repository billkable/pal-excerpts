# Scaling an App with Kubernetes

To gain an understanding of how to scale an application on Kubernetes.

# Learning Outcomes

After completing the lab, you will be able to:

- Explain the difference between vertical scaling and horizontal scaling
- Describe how to horizontally scale your app app running on Kubernetes
- Explain the importance and considerations of horizontal scaling on Kubernetes.

# Scaling Overview

It is important that we configure pre-production environments as close
as possible to our production environment.
The goal is to catch any sort of configuration issues before our
application is deployed to production.

This lesson introduces the concept of _scaling_ in context of
Cloud Native applications.

The term _scaling_ is defined as adding resources to a system to allow
it to handle a growing amount of work.

While scaling might not be necessary on our review environment, most
production applications will need to be _scaled_.

Applications can be scaled in two different ways:

1.  _Vertical scaling_: Each Pod gets more memory, disk
    space, or CPU.
    Workloads that are candidates for vertical scaling are stateful
    workloads, such as caches and databases.

1.  _Horizontal scaling_: The number of Pods serving
    requests is increased.

Cloud native applications use _horizontal scaling_ according to the
[12 Factor Concurrency guideline](https://12factor.net/concurrency)
so _vertical scaling_ is not covered or demonstrated in this course.

## Scenario

Let us pretend that your PAL Tracker web application is already running
in production:

1.  It services 100 "beta" users, at an average of 50
    requests-per-second work rate during the peak usage.

    That means the average work rate for the current application's
    usage profile for each user is **0.5 requests/second/user**.
    This is the *Work rate / User* that you will use as one of the
    variables to help forecast how to scale your app later on.

1.  You have one Pod running in a Kubernetes cluster,
    and the application runs without incident for a couple of months.

1.  Your product owner declares wild success of the initial product
    "beta" release:
    -   The application is ready to go into the wild.
    -   The product estimates the next release will need to
        service 500 concurrent users at its peak usage periods.

    Will 1 Pod be enough?

1.  You need to do some experimental load testing against your PAL
    Tracker app with **500** users at an *Extrapolated work rate*.

    Do the following math to get it:

    `Extrapolated work rate = (# Concurrent users) * ( work rate / user )`

    So...

    `(500 users) * (0.5 request per second / user) => 250 requests per second`

1.  You do the testing with 500 users at 250 total requests per second,
    and find 1 Pod will not be enough -
    it slows down and crashes.

    What do you do?

1.  Through an empirical load testing approach,
    you find you achieve a stable work rate of up to
    **100 requests-per-second** reliably on a single Pod.

    This is your *Baseline work rate / pod* that you will use later on
    to help calculate the number of Pods for the upcoming release.

1.  Calculate the number of Pods you will need to service the 500 users.
    Do that with the following math:

    `#Pods = Round up ((#Concurrent users) * (Per user work rate) / (Baseline pod work rate / pod))`

    So...

    `(500 concurrent users) * (0.5) / (100 requests per second per pod) => 2.5`

    and rounded up to **3 pods**.

    You will use this number in the next section.

# Horizontally scale the number of Pods

When you create a Deployment, Kubernetes also creates another object
known as a
[*ReplicaSet*](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/).

You can view your ReplicaSets by running:

```bash
kubectl get replicasets
```

The purpose of a ReplicaSet is to **maintain a stable set of Pods**
**running our application to help guarantee availability**.

***Important Note:***
***ReplicaSet is a Kubernetes feature that may be used to help achieve***
***higher levels of availability, or horizontal scaling.***
***This lesson focuses on the concept of Scalability.***
***The concept of Availability is covered at more depth in another***
***lesson.***
***It is important to note that Cloud Native applications use Horizontal***
***Scaling as one tool to achieve Stability and Availability characteristics.***
***They also use Rolling Updates or Blue/Green deployments to maintain***
***Availability characteristics during version updates of the application.***

By default the ReplicaSet will create and maintain a single Pod.

1.  Watch the list of currently running Pods:

    ```bash
    kubectl get pods --watch
    ```

1.  In another terminal window, delete the currently running Pod:

    ```bash
    kubectl delete pod POD-NAME
    ```

1.  In the watch window, notice that the existing Pod is terminated and
    a new Pod is created.

    When the ReplicaSet saw that there were no Pods running your
    application, it created a new Pod to maintain your desired state:
    one Pod.

    Now you will tell the ReplicaSet to maintain two Pods to run your
    application.

1.  Edit your `deployment.yaml` to set the number of replicated
    Pods you calculated in the
    [Scenario](#scenario) previous to this exercise:

    ```diff
    spec:
    + replicas: <integer number of pods>
      selector:
        matchLabels:
    ```

1.  Apply the your changes to the development environment in a terminal
    window not running the Pod watch command.

1.  You will see the new Pods created without terminating any existing
    Pods in the watch window.

# Need for empirical testing and production feedback

The [scenario](#scenario) in this lab is naive.
Forecasting capacity requires a lot of empirical load testing scnearios
with a defined *work profile*, the way that your users use your
application.

Designers and developers may predict and define the operations a user
may execute as part of a planned workflow,
but the users may use it in different ways than the designers predicted.

This means that in real production,
getting feedback of how the users are *actually using* the application
is an important consideration for forecasting capacity.

Another consideration includes finding bottlenecks in scaling,
which many times may be at a dependent backing resource,
such as a database.

A factor not considered in this lab is that of *Availability*.
It is necessary to factor into Cloud Native applications running on
modern platforms must be designed for *Disposability*.
Any individual Node or Pod may fail or be shutdown at any time,
for any reason.

# Trade-offs and considerations when scaling on Kubernetes

When you use horizontal scaling on Kubernetes in a *Cloud Native
Application*, you do not tell Kubernetes what Nodes to schedule your
Pods.
It will automatically do that for you.

A consequence is that Kubernetes may schedule containers servicing
different application containers on the same Node.

If a particular Pod takes a lot of resources,
it can compete, constrain, or negatively impact the applications' Pods
stability or performance.

Kubernetes gives several features to protect the integrity of the
infrastructure,
some of which are configured by the Kubernetes Cluster Operators,
and some are optionally configured by you
(the developer configuring your application deployment).

Check out the [Extras](#extras) for more information.

# Assignment

1.  Submit the assignment using the `cloudNativeDeveloperK8sScaling`
    gradle task from within the existing `assignment-submission` project
    directory.
    It requires you to provide the name of your development deployment.

    For example:

    ```bash
    cd ~/workspace/assignment-submission
    ./gradlew cloudNativeDeveloperK8sScaling -PdeploymentName=pal-tracker
    ```

# Clean up

1.  Remove the `replicas` entry in the `deployment.yaml` that you
    created in the
    [Horizontally scaling the number of Pods manually](#horizontally-scale-the-number-of-pods)
    exercise, step 4.

1.  Apply the origin configuration to backout your scaling changes:

    ```bash
    kubectl apply -f k8s/
    ```

1.  You should have no changes to commit and push.

# Learning Outcomes

Now that you have completed the lab, you should be able to:
::learningOutcomes::

# Extras

## Kubernetes ResourceQuotas and Limits

Kubernetes Cluster Operators will normally configure the following to
protect against resource depletion on a Kubernetes Cluster Nodes:

- [Resource Quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/)
- [Limit Ranges](https://kubernetes.io/docs/concepts/policy/limit-range/)

These work on a
[namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
level.

## Resource Requests and Limits

You can be a good citizen by specifying the amount of resources each of
your Pods need to start,
as well as self-imposing limits on each.

- [Managing Container Resources](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers)

*You must understand the requirements and behavior of your*
*application before you attempt to use these features.*
*CPU limits can result in throttling your application,*
*which may slow it down.*
*Exceeding memory limits of Pod will result in the Kubernetes cluster
and the associated operating systems underneath to terminate the
affected Pod.*

-   Learn more about CPU resource request and limit considerations:

    -   [Kubernetes: Make your services faster by removing CPU limits](https://erickhun.com/posts/kubernetes-faster-services-no-cpu-limits/)
    -   [CPU limits and aggressive throttling in Kubernetes](https://medium.com/omio-engineering/cpu-limits-and-aggressive-throttling-in-kubernetes-c5b20bd8a718):
    Some of the bugs in Linux kernel that may already have been fixed,
    but is it contains a good description of Kubernetes and associated
    Linux `cgroup` internals.

## Automatic scaling

In the [Scenario](#scenario) section, we saw a new Cloud Native app go
to production.

After your application grows and matures, it may have a varied work-rate
over time.
You may discover that you need to scale up to handle large amount of
work during certain periods of time,
then scale down during periods when the work rate is low, to save cost.

If you can reliably predict the patterns of variable work-rate,
you can use *Automatic Scaling* (*Autoscaling* for short) features of
Kubernetes, much like modern advanced automobiles have adaptive cruise
control.

Kubernetes provides the
[*Horizontal Pod Autoscaler (HPA)*](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
that can automatically scale the number of Pods.

You can experiment with its default support for CPU-based rules,
but for real world production applications,
you will likely use support for scaling based on
application-provided metrics using
[custom metrics](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/instrumentation/custom-metrics-api.md).

See the [Resources](#resources) section for more details.

# Resources

- [Resource Management](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)
- [Manual Scaling](https://kubernetes.io/docs/tutorials/kubernetes-basics/scale/)
- [Auto Scaling Overview](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [Auto Scaling Walkthrough](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)
- [Controller Manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/)