---
title: About CI/CD
date: 2024-05-13 11:31:00 -0700
categories: [CI/CD]
tags: 
Pin:
---

### What is you want about CI/CD

- Safe 
- Reversible
- Provide no down time

### Deployment

- Rolling Deploymen(Phased Deployment)
- Blue-Green Deployment
- Canary Deployment
- A/B Deployment

### Six continuous integration practices

1. #### Fast builds 

​	High WIP(Work In Progess) leads to problems

2. #### commit small changes

​	Small change makes isolating failure easier

3. #### Don't leave the build broken

4. #### Use a trunk-based development flow

​	**Trunk Diagram**

- No long-running branches
- Developers make small changes
- Check them back in to the trunk multiple times a day

​	**Pros**

- Use a truck-based approach to keep WIP down
- Ensures code is reviewed and checked frequently
- reduces wasteful, error-prone work to merge branches

5. #### Don't allow flaky automated tests, fix them

6. #### The build should return a status, a log, and an artifact

### Five practices for continuous delivery

1. #### Build artifacts only once
2. #### Artifacts should be immutable
3. #### Deploy to a production-like environment
4. #### Stop deploys if a previous step fails
5. #### Deploys should be idempotent

### Questions

1. Which task becomes easier when code is maintained in the cloud?

   having similar preproduction and production environments

2. Artifacts should be `_____`.

   built once and deployed as needed

3. Suppose some of your tests are slow. Which procedures should you select to handle a slow test?

   Utilize time-scheduled testing.

   Apply monitoring to complete some test objectives.

   Employ a non-blocking test.

4. What is the goal for every phase of the continuous delivery process?

   Provide early and rapid feedback

5. Which of the following explains the concept of blue-green deployment?

   There are two identical production environments in which one is live (Blue) and the other is idle (Green).

6. If you are developing end user test scenarios before writing code, then you are engaged in which type of development?

   test-driven development

7. Continuous delivery has all of these benefits except `_____`.

   increasing fragile artifacts

8. Which belief has been disproven by organizations that utilize continuous delivery?

   A high frequency of change leads to a decrease in quality.