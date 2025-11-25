# About the KubeRocketCI Platform

**KubeRocketCI (previously known as EPAM Delivery Platform)** is an **open-source** cloud-agnostic SaaS/PaaS solution for software development, licensed under **Apache License 2.0**. It provides a pre-defined set of CI/CD patterns and tools, which allow a user to start product development quickly with established **code review**, **release**, **versioning**, **branching**, **build** processes. These processes include static code analysis, security checks, linters, validators, dynamic feature environments provisioning. Platform consolidates the top Open-Source CI/CD tools by running them on Kubernetes/OpenShift, enabling web/app development in isolated (on-prem) or cloud environments.

KubeRocketCI, which is also called **"The Rocket"**, is a platform that allows shortening the time that is passed before an active development can start from several months to several hours.

The platform consists of the following blocks:

- The platform is based on managed infrastructure and container orchestration
- Security covering authentication, authorization, and SSO for platform services
- Development and testing toolset
- Well-established engineering process and EPAM practices (EngX) reflected in CI/CD pipelines and delivery analytics
- A set of pre-configured pipelines for different types of applications (polyglot microservices)
- Observability stack

## Features

- Deployed and configured CI/CD toolset ([Tekton](https://tekton.dev/), [ArgoCD](https://argoproj.github.io/cd/), [Nexus Repository Manager](https://help.sonatype.com/en/sonatype-nexus-repository.html), [SonarQube](https://www.sonarsource.com/), [DefectDojo](https://www.defectdojo.org/), [Dependency-Track](https://dependencytrack.org/)).
    
- [GitHub](https://about.gitlab.com/features/)(by default) or [GitLab](https://about.gitlab.com/features/).
    
- [Tekton](https://docs.kuberocketci.io/docs/operator-guide/install-tekton) is a pipeline orchestrator.
    
- [CI pipelines](https://docs.kuberocketci.io/docs/user-guide) for polyglot applications:

https://docs.kuberocketci.io/