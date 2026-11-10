# One-Page Concept Reference

**Status:** Reference created; deeper implementation guide linked below.

## CI/CD Basics

### Short explanation

Continuous Integration automatically builds and checks each proposed change. Continuous Delivery keeps verified software ready for release; Continuous Deployment automatically releases qualifying changes.

### Handy example

~~~text
Pull request -> compile -> unit tests -> integration tests -> package
             -> build image -> staging -> smoke checks -> approval -> production
~~~

### Key point

A green pipeline is useful only when it runs the checks that matter. A command that packages the JAR while skipping tests is not a meaningful quality gate.

## Maven build lifecycle

For a Java backend service, **./mvnw clean verify** is a strong default CI command when the project has configured its tests correctly.

- **compile** compiles production code.
- **test** normally runs unit tests through Surefire.
- **package** creates the JAR/WAR.
- **verify** runs configured verification, including Failsafe integration-test verification when that plugin is configured.

Surefire and Failsafe are not interchangeable. Failsafe must be bound to integration-test and verify; simply naming a class *IT does not guarantee Maven will execute it.

## GitHub Actions

A Java CI job normally:

1. checks out the commit;
2. installs the project's supported JDK;
3. caches Maven dependencies;
4. runs the Maven Wrapper with clean verify;
5. fails the pull request when compilation or required tests fail.

The [Java backend CI/CD guide](https://github.com/RohiniNetekal/Git-Engineering-Workflow-Lab/blob/main/docs/java-backend-ci-cd.md) contains a complete Java 17/21 workflow template, Testcontainers guidance, container-image promotion, OIDC deployment and security controls.

## Security and release design

- Use pull_request for ordinary build/test validation.
- Give build jobs only the permissions they need, usually contents: read.
- Do not expose deployment credentials to untrusted pull requests.
- Build an immutable image once and promote the same digest between environments.
- Prefer short-lived OIDC cloud credentials over long-lived access keys.
- Keep production deployment behind protected environments and approvals.

The Git workflow portfolio repository now has a real Markdown lint workflow. The Java/Maven YAML in the guide is a reusable template for a future runnable Spring Boot project; this learning repository does not currently contain a root Maven build, so no Java build is claimed here.

## Trade-offs and common mistakes

- A Java matrix improves compatibility coverage but increases runner time.
- Testcontainers provides realistic integration coverage but requires Docker and makes tests slower than unit tests.
- Dependency caching improves speed but must never be required for correctness.
- Retrying flaky tests without fixing the race hides defects.
- Rebuilding a different image for production breaks artifact traceability.

## Interview questions

1. What is the difference between CI, continuous delivery and continuous deployment?
2. Why is Maven verify often preferred over Maven package in CI?
3. What is the difference between Surefire and Failsafe?
4. How would you test a Spring repository against a real database in CI?
5. Why should a pull-request build use a read-only token?
6. Why promote the same container digest to staging and production?
7. How does OIDC improve cloud deployment security?

## Practice checklist

- [ ] Explain Maven's lifecycle without looking at notes.
- [ ] Explain why Failsafe is needed for configured integration tests.
- [ ] Read the workflow YAML and explain each permission and trigger.
- [ ] Distinguish a documentation lint run from a Java application build.
- [ ] Describe how a Testcontainers test exercises a database constraint.
- [ ] Explain artifact promotion and deployment rollback.
