+++
title = "Quality Architect Role"
LastModifierDisplayName = "Alain Bouchard"
LastModifierEmail = "abouchard@live.ca"
disableToc = "false"
+++

{{< toc >}}

## Role

The architect is responsible for two main types of high-level activities:

1. Strategic (foundations): take care of the technical vision in the medium to long term (one, two years or even more)
1. Tactics (operations): technical support for teams, problem solving, proof of concept, etc.

## Strategic

1. Define the "Quality Vision", guidelines, technical documents ([ADR], technical designs, terminology/technical glossary, etc.) while taking into account security rules, [PII], etc
1. Evaluate technologies: research, comparisons, proof of concepts, publications, etc
1. Promote best practices for the chosen technologies
1. Mentoring and coaching
1. [Promote quality] during the six stages of software development, [SDLC] or "Software Development Life Cycle": "Planning" -> "Defining" -> "Designing" -> "Building" -> "Testing" - > "Deployment"
1. Assist or lead working groups for specific topics, define the purpose and deliverables, periodic and monthly events
1. Attend or lead workshops for specific topics, define the goal and deliverables, event over a few hours or a few days
1. Audit of the services for test coverage and correlate the data with the problematic services in production (Data-Driven decisions)
1. Analyze the current testing strategy, test characterization, define a plan, promote good testing practices, follow the test pyramid:
    1. [Unit] (solitary and sociable) testing
    1. [Component] (in-process and out-of-process) testing
    1. [Contract] testing
    1. [Integration] (narrow and broad) testing
    1. UI End-to-end ([E2E]) and API E2E ([Subcutaneous])
    1. [User journey] tests (and [Synthetic Traffic] tests)
    1. [Non-functional] tests: load, scalability, webvital metrics [LCP], chaos/robustness, etc
1. Alignment on a test strategy with the deployment procedure ([CI]/[CD]) and a [branching] procedure (gitflow, githubflow, etc), [blue/green deployment], etc
1. Participates in the recruitment process for the “quality” aspects

## Tactics

1. Apply the test strategy: develop test projects, add the projects to the process (build, [CI]/[CD], [cluster/kubernetes]. etc), document, etc
1. Support test development: support QAs and developers who add tests, PR reviews, etc
1. Add test coverage: JavaScript (e.g., cypress.io, etc), python/pytest, java/rest-assured, etc
1. Addition of [Synthetic Traffic] tests and verification of customer flows and scenarios (i.e., customer flows)
1. Analyze system performance (e.g., load tests) and according to defined standards (i.e., *"Given the data load X and the number of users Y, an API should respond within Z seconds"*), [LCP] (i.e., Largest Contentful Paint) (e.g., *"a page should be considered usable in less than 2.5 seconds"*)
1. Using observation tools to understand issues: [datadoghq], [splunk/signalfx], [prometheus], ​​[grafana], etc
1. Collaborates horizontally with the different “domaines” of the company and suppliers to improve the overall quality of the company

[ADR]: https://adr.github.io
[blue/green deployment]: https://martinfowler.com/bliki/BlueGreenDeployment.html
[branching]: https://medium.com/@vafrcor2009/gitflow-vs-trunk-based-development-3beff578030b
[CD]: https://www.atlassian.com/continuous-delivery/continuous-deployment
[CI]: https://www.atlassian.com/continuous-delivery/continuous-integration
[cluster/kubernetes]: https://www.vmware.com/topics/glossary/content/kubernetes-cluster.html
[component]: https://martinfowler.com/articles/microservice-testing/#testing-component-introduction
[contract]: https://martinfowler.com/bliki/ContractTest.html
[datadoghq]: https://www.datadoghq.com/
[E2E]: https://martinfowler.com/articles/microservice-testing/#testing-end-to-end-introduction
[grafana]: https://grafana.com/
[integration]: https://martinfowler.com/articles/microservice-testing/#testing-integration-introduction
[LCP]: https://web.dev/lcp
[Non-functional]: https://www.guru99.com/non-functional-testing.html
[PII]: https://en.wikipedia.org/wiki/Personal_data
[prometheus]: https://prometheus.io/
[Promote quality]: https://github.com/AlainBouchard/engineering-notes/blob/master/how-to-promote-the-quality-best-practices.md
[SDLC]: https://www.tutorialspoint.com/sdlc/sdlc_overview.htm
[splunk/signalfx]: https://www.splunk.com/
[Synthetic Traffic]: https://www.datadoghq.com/knowledge-center/synthetic-testing
[subcutaneous]: https://martinfowler.com/bliki/SubcutaneousTest.html
[unit]: https://martinfowler.com/articles/microservice-testing/#testing-unit-introduction
[user journey]: https://martinfowler.com/bliki/UserJourneyTest.html
