# READ ME

This package contains excerpt of labs from former PAL for Developers on
Kubernetes class, as well as example codebase for the Application
Continuum Talk, and reference links.

## Spring Boot 2.4.x Actuator, Kubernetes and containers usage/integration features

- [Health Groups](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-health-groups)

- [Kubernetes Probes](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-kubernetes-probes)

- [Application Availability](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-application-availability)

- [Application Availability State Management](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-application-availability-managing)

- [Container features](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-container-images)

## Kubernetes availability probes labs

The first 4 lessons cover basic kubernetes connectivity and arch
concepts, and assumes you know how to build containers and deploy to K8s.

The following labs use the two imbedded codebases (local git repositories
with start and solution points for each lab):

- [`pal-tracker` Spring Boot codebase](https://github.com/platform-acceleration-lab/pal-tracker-k8s.git)
- [`k8s` resource configuration](https://github.com/platform-acceleration-lab/pal-tracker-k8s-deployment.git)

In each of the labs, you are directed in the *Getting Started* sections
to `git cherry-pick` the start points.
Consider using `git reset --hard` to the start and/or solution points
given you are not working from the beginning of the course.

-   [05 - Probes](./05-probes.md)

-   [06 - Rolling Updates](./06-rolling-updates.md)

-   [07 - Scaling](./07-scaling.md)

-   [08 - Liveness Probes](./08-liveness-probes.md)

## Tanzu developer workshops

- [Cloud Native Development Fundamentals Deployment Practices](https://tanzu.vmware.com/developer/workshops/cnd-deploy-practices/)

- [Cloud Native Development Fundamentals Configuration Practices](https://tanzu.vmware.com/developer/workshops/cnd-config-practices/) -> NOTE, this is not published yet at
the time I am sending this, but should be up mid-week.

## Cloud Native Fundamentals Slide references

-   Introduction - Lecture #1
    ([Slides](https://docs.google.com/presentation/d/1Bg9Ra_vMIzq4Gq5hfaZ0jopmKj1obvwov-Rk4xZbyBY/present#slide=id.g7ea4a9dfdf_0_1813))

-   Personas - Lecture #2
    ([Slides](https://docs.google.com/presentation/d/1T5Tjcqf0R-njyGSH3AQ7KqGdkX-co-GM0jlVAjRMk1M/present#slide=id.g7ea4a9dfdf_0_1813))

-   Guidelines and Practices - Lecture #3
    ([Slides](https://docs.google.com/presentation/d/1QaBOfQK5_3DDNO9xi3I2N3gczM8243fUhvN7bPpLlcI/present#slide=id.gb51335f931_1_156))

-   Platform Concepts - Lecture #4
    ([Slides](https://docs.google.com/presentation/d/1fepud2V36GcsUNrUA1Aaz23q0CRFqvSs_a0yr5ehUTc/present#slide=id.g7ea4a9dfdf_0_1813))

-   Blocking web apps - Lecture #5
    ([Slides](https://docs.google.com/presentation/d/1lMcWJLcnp3w3qf5UAa2CWnd_VPbH15rMri6jtoolNAs/present))

-   Containerize an app - Lecture #6
    ([Slides](https://docs.google.com/presentation/d/1RrDgRkuM5bSPHV6P5uvkvb9AFn10AMYq2V67gBShjUQ/present))

-   Accessing your platform - Lecture #7
    ([Slides](https://docs.google.com/presentation/d/1ctnmoRcgbcVyBndFBDr00HtE23ky8GSqFIgKkRmo2Tw/present))

-   Deploying an app - Lecture #8
    ([Slides](https://docs.google.com/presentation/d/1mzZurB3sDo-7_Rj0p6RKv_gajg1rvFitPUd02Yvr-f8/present))

-   Configuration Dependencies Introduction - Lecture #10
    ([Slides](https://docs.google.com/presentation/d/1VyNcZ_UmkEVeVrUALAkXTKYRuZEMwFepQdBct7j-Urk))

-   Configuration Dependencies - Lecture #10a
    ([Slides](https://docs.google.com/presentation/d/17mkMH5tD-xX4CZ41iDrYFggiTuM_CIEnnjYWR3Kg8MA))

-   Platform Behaviors and Contraints - Lecture #11
    ([Slides](https://docs.google.com/presentation/d/1skgQGPm2sBQM59bVMYEhh2seMpQd8HNekluYxJoRyMI))

-   Applications, Deployments, Constraints - Lecture #12
    ([Slides](https://docs.google.com/presentation/d/1ihTQrSc5Jpv01SOD18C9TAjknha3sVabGr2zZ-k0PMg))

-   Environment Configuration - Lecture #13
    ([Slides](https://docs.google.com/presentation/d/1JxtHb8i9R1_wndlc_zKsyZooKkgpel2gj8U7Z7SoBs0))

-   Environment Variables and Backing Services - Lecture #14
    ([Slides](https://docs.google.com/presentation/d/1V6GT1FHzCVe4W0-7UASC7F84WghlK77M7PnHBBOBRC0))

-   Configuration synchronization - Lecture #15
    ([Slides](https://docs.google.com/presentation/d/1R_Vi2VOKhXwomhegqWmNjKnHU-yDPdnW-vGs-FJ_3-Y))

## Application Continuum

- [Site](https://www.appcontinuum.io/)
- [Repo with example Kotlin code](https://github.com/continuumcollective/application-continuum)
- [Presentation](https://deck.appcontinuum.io/)
- [`pal-tracker-distributed` Java codebase at `HEAD` (root commit)](https://github.com/platform-acceleration-lab/pal-tracker-distributed)

## Tanzu Application Services Fundamentals Slides

[TAS Fundamentals](https://docs.google.com/presentation/d/17NxY9m73TDW2aiXvCDMeV7y74ox2F4bnV8ye921c0VQ)

Sections of relevance:

[PAL Tracker Distributed deployment visualization](https://docs.google.com/presentation/d/17NxY9m73TDW2aiXvCDMeV7y74ox2F4bnV8ye921c0VQ/edit#slide=id.gb5c38a7169_0_1460)

