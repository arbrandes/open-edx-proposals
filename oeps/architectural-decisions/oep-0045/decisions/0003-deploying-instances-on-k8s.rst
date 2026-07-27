Deploying Open edX on Instances Kubernetes
##########################################

Status
******

Draft


Context
*******

Hosting Open edX instances on Kubernetes requires a complex infrastructure setup, especially when multiple instances are involved. Decisions made during cluster setup can directly affect the performance, latency, and reliability of those instances

The `decision 0002`_ elaborates on the need for community-maintained components to reduce this complexity. Although the Terraform modules and Helm chart support a modular setup already removing a significant part of the complexity, they do not fully abstract that away. Furthermore, none of these components are addressing the significant challenge of deploying instances on Kubernetes.

Open edX commercial providers are addressing this challenge with purpose-made solutions that are mostly not compatible with each other, although they are built using some community-maintained and provided components.

.. _decision 0002: https://docs.openedx.org/projects/openedx-proposals/en/latest/architectural-decisions/oep-0045/decisions/0002-openedx-hosting-infrastructure-on-k8s.html


Decision
********

The `Large Instances Working Group`_, mostly consist of the members of commercial providers, maintains the Terraform modules and Helm chart described in decision 0002, as well as the growing ecosystem of related open-source deployment tools.

For operators deploying Open edX on Kubernetes, these resources (`Drydock`_, `Picasso`_, `ArgoCD`_, `Argo Workflows`_ and `GitHub Actions`_) provide flexible reference architectures.
We recognize that Open edX deployments vary widely in scale and infrastructure. Operators are free to blend these community tools with their own existing stack, and we encourage providers to share new deployment code, modules, or templates with the community.

Because Open edX intentionally supports diverse infrastructure needs, none of these tools are mandatory. Instead, the community focuses its efforts on ensuring these reference implementations and provider templates like OpenCraft's `Launchpad`_, are maintained to serve as reliable blueprints.

.. _Large Instances Working Group: https://openedx.atlassian.net/wiki/spaces/COMM/pages/3655008783/Large+Instances+Working+Group
.. _GitHub Actions: https://github.com/features/actions
.. _Drydock: https://github.com/eduNEXT/drydock
.. _Picasso: https://github.com/eduNEXT/picasso
.. _ArgoCD: https://argo-cd.readthedocs.io/en/stable/
.. _Argo Workflows: https://argoproj.github.io/workflows/
.. _Launchpad: https://github.com/open-craft/launchpad-cluster-template


Consequences
************

Introduction
============

The following considerations describe the trade-offs and best practices for deployments following the community reference architecture outlined in this decision. They are not requirements for deploying Open edX on Kubernetes, but recommendations for operators who choose to adopt the community-maintained tooling ecosystem.

Build Pipeline Considerations
=============================

In order to build Open edX instance images with Tutor, the CI/CD pipelines should be executed. The cluster template is going to use Picasso to simplify the image building process, though this binds the rendered clusters to GitHub Actions.


Deployment Pipeline Considerations
==================================

The deployment of instances are going to be handled by ArgoCD that reads the Kubernetes manifests rendered by Tutor and pushed to the cluster repository by GitHub Actions. Although this keeps continuous deployment convenient, easy to oversee, and ensures that every change can be audited, secrets are ending up in the version control system.

This is the result of some limitations coming from Tutor-rendered Kubernetes manifests.


Best Practices
==============

In order to keep a healthy state for the cluster template, these best practices should be followed:

* All infrastructure dependency should be coming from the community-maintained and provided Terraform modules and Helm charts, laid out in decision 0002.
* Per-instance resource provisioning (e.g., database users) should be as much provider-agnostic as possible.
* No company- or Open edX provider-specific logic should live in the cluster template or rendered cluster.
* The instance deployment should be handled by cloud provider-agnostic tooling.
* The CI/CD pipelines should do only the bare-minimum needed for instance setup.
* No fragile or dangerous operations should be performed by the CI/CD pipelines except the instance deprovisioning flow.
* The operators should handle the infrastructure modifications from their computer rather than from CI/CD pipelines.
* Infrastructure provisioning should be done by the 3rd-party CLI tooling (such as `argo`, `tofu`/`terraform`, etc.) or by the scripts provided by Launchpad that is wrapping these 3rd-party tools.


Change History
**************

2026-07-03
==========

* Document created
