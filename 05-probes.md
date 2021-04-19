# Configuring Availability State Probes

You will use
[Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready.html)
features to allow a runtime platform to query the state of your
application in this lab.

You will exercise the readiness and liveness state endpoints for your
Spring Boot application.

# Learning Outcomes

After completing the lab, you will be able to:

- Explain the value of readiness probes during application startup
- Explain the value of liveness probes for running applications
- Explain how Actuator implements readiness and liveness probe features.

# Get started

Before starting the lab,
run the cherry pick to get some new code:

```bash
git cherry-pick availability-probes-start
```

This added a `PalTrackerFailure` class, which you will use in the
*Liveness* section of the lab to demonstrate Spring Boot
`AvailabilityState`, and failing test for detection of the availability
probes, which you will make pass.

# Availability Probes

An availability probe is contract between an application and a runtime
platform.

The application instances expose different availability types to the
platform, and the platform can take a specific actions based on those
types.

The two most common types are:

- readiness probe
- liveness probe

Spring Boot 2.3.x and above allow your application to:

- Define rules for availability types
- Expose availability probe endpoints via actuator health endpoints

## Enable probes locally

When running locally on a development workstation, availability probes
are not enabled.
You will need to enable them.

1.  Add the following environment variable to the `bootRun.environment()`
    and `test.environment()` in the `build.gradle` file:

    `"MANAGEMENT_HEALTH_PROBES_ENABLED": true`

1.  Restart the `pal-tracker` application on your development workstation.

1.  Verify a request to the `actuator/health` endpoint has a 200 HTTP
    response and an *up* status:

    ```bash
    curl -v localhost:8080/actuator/health
    ```

1.  Verify the `readinessState` and `livenessState` entries are both
    present in the response of the `health` endpoint with *up* status.

1.  Verify a request to the `actuator/health/readiness` endpoint has a
    200 HTTP response and an *up* status:

    ```bash
    curl -v localhost:8080/actuator/health/readiness
    ```

    This is the `readiness` probe endpoint.

1.  Verify a request to the `actuator/health/liveness` endpoint has a
    200 HTTP response and an *up* status:

    ```bash
    curl -v localhost:8080/actuator/health/liveness
    ```

    This is the `liveness` probe endpoint.

## Readiness

The readiness state, the ability to service requests, is published by
your application through a readiness probe endpoint.
The platform can use the state of the readiness probe to determine when
to route requests to the application.

Spring Boot exposes the probe via the `actuator/health/readiness`
endpoint.
The default readiness probe measures that the Spring web container
is running.

What if your application has an external dependency on a *backing
service* and the state of that dependency needs to be considered in the
readiness state of the application?
One such example of this, that you will see later, is that of a
relational database.

1. Review the code for the `BackingService`:

    {{codebase-file codebase="pal-tracker" path="src/main/java/io/pivotal/pal/tracker/BackingService.java" ref="availability-probes-solution" lang="java" hidden="true"}}

    Notice that the backing service has an initialization delay the first
    time it is accessed.

1. Review the code for the `BackingServiceHealthIndicator`:

    {{codebase-file codebase="pal-tracker" path="src/main/java/io/pivotal/pal/tracker/BackingServiceHealthIndicator.java" ref="availability-probes-solution" lang="java" hidden="true"}}

    Notice that the state of the health indicator is based on a
    successful ping of the backing service.

You can override the default behavior of the readiness probe to use
existing health indicators through the Spring Boot
[Health Groups](https://docs.spring.io/spring-boot/docs/2.3.1.RELEASE/reference/html/production-ready-features.html#production-ready-health-groups)
feature.

First, verify the default readiness probe behavior DOES NOT query the
`BackingServiceHealthIndicator` and the associated backing service
initialization:

1.  Restart `pal-tracker` application on your development workstation.

1.  Verify a request to the `readiness` endpoint has a 200 HTTP
    response and an *up* status:

    ```bash
    curl -v localhost:8080/actuator/health/readiness
    ```

1.  Verify you do NOT see the following on the `pal-tracker` console
    output:

    ```bash
    ... io.pivotal.pal.tracker.BackingService    : Initializing Backing Service...
    ... io.pivotal.pal.tracker.BackingService    : Backing Service is initialized, ready to service requests...
    ```

    This verifies that the `BackingServiceHealthIndicator` is not
    included in the default readiness probe.

Next, verify that the `health` endpoint does query the
`BackingServiceHealthIndicator` and the associated backing service
initialization:

1.  Verify a request to the `health` endpoint has a 200 HTTP
    response and an *up* status:

    ```bash
    curl -v localhost:8080/actuator/health
    ```

1.  Verify that you DO see the following on the `pal-tracker` console
    output:

    ```bash
    ... io.pivotal.pal.tracker.BackingService    : Initializing Backing Service...
    ... io.pivotal.pal.tracker.BackingService    : Backing Service is initialized, ready to service requests...
    ```

    This verifies that the `BackingServiceHealthIndicator` is included
    in the default health endpoint and that the `BackingService` is
    pinged as part of that health check.

Now, we want to override the default readiness check to include the
`BackingServiceHealthIndicator`:

1.  Add the following to the `bootRun.environment()` configuration in
    the `build.gradle` file:

    `"MANAGEMENT_ENDPOINT_HEALTH_GROUP_READINESS_INCLUDE": "backingService"`

1.  Restart `pal-tracker` application on your development workstation.

1.  Verify a request against the readiness endpoint with a 200 HTTP
    response and an *up* status:

    ```bash
    curl -v localhost:8080/actuator/health/readiness
    ```

1.  Verify that you DO see the following on the `pal-tracker` console
    output:

    ```bash
    ... io.pivotal.pal.tracker.BackingService    : Initializing Backing Service...
    ... io.pivotal.pal.tracker.BackingService    : Backing Service is initialized, ready to service requests...
    ```

    This verifies that we have added the `BackingServiceHealthIndicator`
    to the readiness probe.

    This will insure that the `BackingService` is healthy before the
    platform will route traffic to our application.

You can use autoconfigured resource health indicators such as JDBC
Datasource, RabbitMQ, Kafka, or NoSQL DBs,
to verify backing services as prerequisites for your Spring Boot
application to be ready to service requests.

## Liveness

The liveness state, the ability to answer requests in a timeley fashion
or the signal that the application is not having issue, is published by
your application through a liveness probe endpoint.
The platform can use the state of the liveness probe to destroy and
re-create application instances that can not adequately service requests.

Spring Boot exposes the probe via the `actuator/health/liveness`
endpoint.
By default the liveness probe defaults to the Spring web container
being up.

Let's pretend that our application can no longer service the requests
reliably.
Perhaps the app encounters a fatal exception from which it cannot recover,
or there is a *persistent* failure of a backing resource.

Unfortunately the `BackingServiceHealthIndicator` is not suitable to
use for the liveness probe because it also includes *transient* failures
of the backing service.
We will talk about this further in
[Special Considerations](#special-considerations).

The cherry-picked code provided a new tool that you will use to
demonstrate a broken liveness state in your `pal-tracker` application:

1.  Review the `PalTrackerFailure` actuator management endpoint:

    {{codebase-file codebase="pal-tracker" path="src/main/java/io/pivotal/pal/tracker/PalTrackerFailure.java" ref="availability-probes-start" lang="java" hidden="true"}}

    Notice that it publishes events to simulate breaking the liveness
    state, and how it allows us to simulate a recovery.

1.  Restart `pal-tracker` application on your development workstation.

1.  Verify a request against the liveness endpoint with a 200 HTTP
    response and an *up* status:

    ```bash
    curl -v localhost:8080/actuator/health/liveness
    ```

1.  Simulate you application becoming unhealthy:

    ```bash
    curl -v -XPOST localhost:8080/actuator/palTrackerFailure
    ```

1.  Verify a request against the liveness endpoint with a 503 HTTP
    response and an *down* status:

    ```bash
    curl -v localhost:8080/actuator/health/liveness
    ```

    The liveness endpoint will reflect the current availability state
    of your application.

1.  Simulate you application becoming healthy:

    ```bash
    curl -v -XDELETE localhost:8080/actuator/palTrackerFailure
    ```

### Special Considerations

It may be tempting to use the autoconfigured backing resource Health
Indicators to override the liveness probe of your application.

[*Do not do this!*](https://docs.spring.io/spring-boot/docs/2.3.1.RELEASE/reference/html/spring-boot-features.html#boot-features-application-availability-liveness-state)

Be aware that the health indicators for most backing services show a
near realtime state of connections to a resource, and as such, may
capture *transient* and *persistent* failure events.

You should generally not use transient failures alone to calculate
the liveness state of your application, as this may cause the platform
to mistakenly destroy and re-create application instances.
This can lead to cascading failures in your system as application
instances are destroyed and re-created due to trainsient failures
of backing resources.

You should think deeply about what defines the rules of your application
to announce to the platform that it gives up, and define those rules in
code, or a dedicated health indicator.

The code example in the `PalTrackerFailure` is an example of one way to
define a Liveness failure event;
however,
it is not a realistic one.

Designing proper liveness state calculation may require significant load
testing or production experience to understand the "cracks" in the
application characteristics to properly design and tune around the
availability probes.

You can also define those rules in a custom Health Indicator,
and an associated liveness health group override like we did with the
readiness probe example.

# Submit this assignment

1.  Run the build to make sure that the tests are passing.

1.  Commit and push your changes to Github.

1.  Submit the assignment using the
    `cloudNativeDeveloperK8sAvailabilityProbes` Gradle task.

    **Notice that it requires you to add the `/actuator` root endpoint.**

    For example:

    ```bash
    cd ~/workspace/assignment-submission
    ./gradlew cloudNativeDeveloperK8sAvailabilityProbes -PserverUrl=http://localhost:8080/actuator
    ```

# Extras

If you have additional time:

-   Experiment with the `BackingServiceFailure` endpoint to simulate
    a failed, then recovered readiness state.

-   Consider trying a Health Group override using by building a custom
    `PalTrackerHealthIndicator` that handles the Liveness detection
    logic instead of publishing Liveness availability event within the
    `PalTrackerFailure`.

# Resources

- [Health Groups](https://docs.spring.io/spring-boot/docs/2.3.1.RELEASE/reference/html/production-ready-features.html#production-ready-health-groups)
- [Built in liveness and readiness probes](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-kubernetes-probes)
- [Application Availability Features](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-application-availability)