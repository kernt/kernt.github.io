## What is it?

_Pipelines_ (or _actions_ in _GitHub_) enable us to continuously and conditionally run our _E2E tests_. We can decide when and whether tests are run; since E2E tests usually take some time, that’s a good thing.

## So many possibilities; here are some of them.

**1. Setup your pipelines within your git project**

- GitLab: “gitlab-ci.yml” file in your _root folder_
- GitHub: “.github/workflows/*.yml” files

**2. Scheduled test runs  
**Tests can run scheduled and independently from other jobs. Especially for headed or more extensive tests, this makes sense since they often take quite some time to run.

**3. Merge / Pull Requests  
**Run tests on creating a merge request and enable merging only on success. This way, only tested code will end up in your code base.

**4. Trigger tests manually  
**GitHub and GitLab let you run pipelines manually; you can even set variables and conditions to determine which pipelines should run.

**5. Test reports  
**Test reports can be displayed on the platforms (GitHub/GitLab) for easy access and an overview of your tests. You can download HTML reports or automatically upload them to your server to have an accessible resume for your client.

**6. Notifications  
**GitLab/GitHub enables you to get notified via different channels, e.g., slack, email, etc., about your pipelines' success. This way, you are actively updated about issues with your software.

Documentation on setting up pipelines:  
[docs.github.com/en/actions  
](https://docs.github.com/en/actions)[docs.gitlab.com/ee/ci/pipelines](https://docs.gitlab.com/ee/ci/pipelines/)

[](https://docs.gitlab.com/ee/ci/pipelines/)

![](IiYXCyAg_RVawPlnnuZylA.webp)

## Why do we recommend it?

Clean and continuously tested code helps everyone. It makes life easier for the client and the developer and ensures a stable user experience. Automatic scheduled or conditional tests alert you timely of issues in your application and external code and give you peace of mind.

**Why run E2E tests with pipelines in GitHub/GitLab?  
**Using those pipelines instead of third-party solutions makes sense if your project is already set up in GitHub/GitLab. Benefits: Everything from a single source: _code_, _pipelines_, _tests,_ and _reports_.

Pipelines in either GitHub or GitLab offer you all needed tooling and options for running your E2E tests, e.g.:

- setup and maintaining of pipelines/ actions inside your project
- scheduled pipelines
- notifications
- disabling merge requests on failure to ensure that defective code does not enter your codebase
- overview of jobs and reports to identify issues and enable an easier way to fix them
- easily accessible environment to view test results for developers and possibly clients

![](tf_q7Ie9O1tWhB1reiiapw.webp)

## When do we recommend it?

Almost always. Suppose you have set up your CI/CD separately or are not using GitLab/GitHub. In that case, it’s probably easier and more aligned to use the existing infrastructure instead of a separate one in GitHub/GitLab.

But even if you are not using either GitHub or GitLab, it is still an option and can make sense to set up your E2E tests in a GitLab/GitHub project separate from your code. This depends on your project, your infrastructure, and your preferences.

Headed tests, multi-device tests, or those that take a lot of time often can hinder your working process if you set them up on merge/ pull requests. They should run scheduled and timed outside of busy work and deployment times.

## How do we use it?

We have been creating software for over 27 years, and testing and test automatization is ingrained into our daily working process.

Author: Katja Krone  
Illustrations: [Kai Sinzinger](https://www.instagram.com/kai_sinzinger/)

This article is part of a series called #24TechBites. To sweeten the remaining time until Christmas, we aim to inform you about current technologies, inspire you or give you an opinionated review of software trends — all in the form of 24 small daily surprises in the German tradition of an “Adventskalender”.

**To enjoy all the other #TechBites and find out more about us and our advent calendar, click** [**#24TechBites**](https://medium.com/das-b%C3%BCro-am-draht/24techbites-58be847f5f8d)**!**

![](https://miro.medium.com/v2/resize:fit:700/1*64GPPicd7JHvQUGMdBnQog.png)

## About

[_Büro am Draht_](https://t.dasburo.io/b78f46) is a Berlin-based consultancy helping our clients to build resilient and adaptable digital platforms to support today’s business requirements and even launch tomorrow’s business models we might not yet foresee. Hence, we work closely together at every stage of the digital transformation process — from digital strategy to solution design & development to operational support.

Our agile approach to developing versatile and scalable solutions ensures that our clients consistently deliver engaging and personalized customer experiences. To learn more about how we help companies across industries visit:

[dasburo.com](https://t.dasburo.io/b78f46)  
[LinkedIn](https://t.dasburo.io/6ce68e)

Want to hear from our experts on a regular basis?  
Sign up for our newsletter (published in _German_) [_here_](https://dasburo.com/newsletter?place=blog).