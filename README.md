# Activities to Showcase OpenShift Concepts

This repository document a series of activities to test or demo a variety of OpenShift or OKD functionalities or capabilities. It is intended to be used with OpenShift 3.11 and higher. Some of these activities should work with prior versions but they have not been tested for it.

If you find a error or bug, feel free to submit a PR to this repo. If there is a functionality or demo you will like to see included, feel free to submit a request for it.

    NOTES:
    
    There are many OpenShift demos & labs. This is not a replacement of those.
    
    The goal of this repo is to document activities or tasks to highligh various features or concepts in OpenShift.
    
    The initial set of demo, labs or activities in this repo are the result of moving the demos documentation out of the _podcool_ repo into this dedicated repo.

# Demos & Activities
These demos and activities use the ``podcool`` demo application which can be found at [https://github.com/williamcaban/podcool](https://github.com/williamcaban/podcool)

ACTIVITIES:
- [Deploying an App (GUI)](deploy_app_gui.md) This lab demonstrate how to deploy an app in OpenShift using the _Developer Console_.
- [Deploying an App (CLI)](deploy_app_cli.md) This lab demonstrate how to deploy an app in OpenShift using the _OpenShift CLI client_.
- [Pod Resiliency](pod_resiliency.md) This lab demonstrate the self-healing capabilities available to applications deployed in OpenShift
- [Deployment Strategies](deployment_strategies.md) This lab demonstrate how to use build strategies like: s2i and docker to onboard an application
- [Splitting Traffic](routes_splitting_traffic.md) This lab demonstrate how to use OpenShift router to split traffic across different versions of an application.
- [CI/CD Pipelines](https://github.com/williamcaban/podcicd) This activity shows a multi-project CI/CD pipeline
- [ServiceMesh](https://github.com/williamcaban/openshift-servicemesh) Simplify installation of OpenShift ServiceMesh
- [Simulating Webhooks](webhooks_simulation.md) This lab simulate rebuilds triggered by webhooks.

DEMOS:
- [Quay Registry Demo](quay_registry_overview.md) This demo ilustrates the secuirty scanning functionalities of Quay.io or _Quay Enterprise_.

Comming soon:
- Multus
- CNV (KubeVirt)
