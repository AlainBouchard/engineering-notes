+++
title = "How to promote the Quality Best Practices"
chapter = false
LastModifierDisplayName = "Alain Bouchard"
LastModifierEmail = "alain.bouchard@appdirect.com"
disableToc = "false"
+++

{{< toc >}}

## How to promote the Quality Best Practices

Two ways to promote the quality best practices are: *Influencing* the teams or to *enforce* the best practices. A periodical team *reality check* about the current practices is the beginning of the process. A team can easily live in their own *Ivory Tower* or to be *biased* by this statement: *"We always did it that way so why changing it?"*.

## Influence or evangelize

- Discussions: gather information about current pain points (retrospectives and lessons learned)
- Involve QA Specialists in all SDLC (or [Software Development Life Cycle](https://www.tutorialspoint.com/sdlc/sdlc_overview.htm))
- 1:1 meetings to get information and expose point of views, share ideas and explain the best practices: it may be easier to convince on person than a group
- Talk to the teams
- Provide existing articles and blog posts
- Tech talk to expose the vision
- Create training material: guidelines, processes, videos (e.g.: [Pluralsight](https://www.pluralsight.com/), etc.)
- Involve people in quality solutions:
  - Organize Workshops (on short term) so that the targeted public becomes active participants
  - Organize Work Groups (on long term) so that the concerned people become active participants
- To use data as leverage; more information is available in the [data-driven decision](#data-driven-decisions) section

### Data-Driven decisions

To show metrics of the current situation (or challenge the "no metrics" or lack of metrics).  It is the also called ["Shift-Right" approach](https://github.com/AlainBouchard/testing-vision#shift-right-approach) in the Software Quality world.

- End-users or Customers survey can be used (e.g.: use a post-transaction experience survey on the mobile app or web site)
- White-Label partner surveys
- Use production dashboards (examples: [Datadog](https://www.datadoghq.com/) or [SignalFX](https://www.splunk.com/en_us/investor-relations/acquisitions/signalfx.html) metrics) and performance tools (e.g.: [WebVitals](https://web.dev/vitals/), etc.) to monitor [APIs](https://www.datadoghq.com/blog/monitor-api-performance-with-runscope-and-datadog/), [SQLs](https://www.datadoghq.com/blog/sql-server-performance/), etc.
  - Daily performance emails can be sent to the engineering teams
- NPS and CSAT surveys: [NPS vs CSAT definitions](https://www.questionpro.com/cx/csat/nps-vs-csat.html)
- Real Customers Journey (traces of the end-users through the applications)
- Use support CIM dashboards: customer tickets, production tickets, ticket aging/bug fix time, tickets by services or teams, RCA (find the issues caused by quality process)
  - CIM stands for Customer Interaction Management
  - A way to generate CIM data is to use a tool like [eazy bi](https://eazybi.com/features)
- Correlation between test coverage (static code) and production issues
  - It is also *possible* to get test coverage from a live service using a tool like [Sealights](https://www.sealights.io)
- Compile the functional testers (manual tests) created Tickets/JIRA vs current team or service test coverage
- Create Success Stories: start with one team and use the before/after statistics
- GIT statistics (e.g. with a tool like [GitPrime (now Pluralsight Flow)](https://www.pluralsight.com/product/flow)) in order to correlate bugs/tickets and developers stats (e.g.: number of commits, active days, etc.)
  - I.e.: low stats may be caused by developers not working on features or test automations because they are working on manual tests or production support
- Correlate the release cycle (how often the team releases or time to market) and test coverage; better/reliable test coverage and test regression suite will shorten the release cycle.
- Gather statistic about how much time it takes to refactor an existing feature?  Better test coverage will result in faster refactoring

## Enforce

- Contact the best practices stakeholders and ask for support (I.e.: to push the best practices downward)
- To fix deadlines (must be adopted by specific dates)
- Enforce with Gates, checklists, DoD, etc.
- Set Quality specific OKRs
- Assign Quality trainings
