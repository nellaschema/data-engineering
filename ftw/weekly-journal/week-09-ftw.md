# Week 09 - From Development to Production

**Weekly Session - September 19, 2026**

Last lecture with Sir Myk.

This session made me realize that writing code that works is only one part of data engineering. The harder part is making sure that code can **reliably run in production even when you're no longer actively supporting it.**

## What does a reliable data pipeline mean?

One of the main questions from the session was:

> What does a reliable data pipeline mean?

My first instinct would probably be "a pipeline that works."

But that's not enough.

A production pipeline needs more than correct SQL and a working pipeline. It needs to be:

* repeatable
* traceable
* testable
* recoverable
* maintainable

The goal is to build something that **continues to work safely over time.**

---

## Development → Production

> "It's easy to code, it's hard to push it to production."

Normally, we might write and experiment with code in VS Code instead of immediately committing changes or using Databricks resources.

Each environment has a different responsibility:

```text
VS Code
   ↓
Experiment / Develop
   ↓
GitHub
   ↓
Version Control / Collaboration
   ↓
Databricks
   ↓
Run the Pipeline
```

VS Code can act as our sandbox for experimentation.

GitHub gives us version control and a shared place for collaboration.

Databricks is where the actual pipeline can run.

The important idea for me was:

> **Production becomes easier when each environment has a clear responsibility.**

Without a controlled path for changes, production can become chaotic, especially when multiple developers are making changes at the same time.

---

## CI/CD

CI/CD isn't just about "automatically deploying stuff."

It's a **process**.

Sir Myk emphasized:

> "Without an established process, you automate an error."

That one stuck with me.

Automation doesn't automatically make something reliable. If the process is wrong, automation just makes the wrong process happen faster and more consistently.

Another important principle:

> "If you can do it manually, you can automate it."

But before automating the workflow, we need to understand the workflow itself.

### CI: Continuous Integration

CI is where we check that changes are safe to integrate.

This can include things like:

* Code checks
* Data quality checks
* Validation
* Testing

### CD: Continuous Deployment

CD is the controlled process of getting validated changes into the environment where the pipeline runs (Databricks).

The exact implementation can vary but we need to have a **standard process that everyone follows.**

Everyone on the team should understand:

```text
Change
  ↓
Check
  ↓
Test
  ↓
Version
  ↓
Deploy
```

**Traceability** is a big deal in data engineering.

If something breaks in production, we should be able to answer:

* What changed?
* Who changed it?
* When was it changed?
* Which version was deployed?
* What happened after deployment?

---

## The Deployment Loop

Production engineering isn't:

```text
Write → Deploy → Done
```

It's a controlled feedback loop:

```text
CHANGE
   ↓
TEST
   ↓
VERSION
   ↓
DEPLOY
   ↓
VERIFY
   ↓
DIAGNOSE
   ↓
FIX
   ↓
REDEPLOY
   ↺
```

This changed how I think about deployment, which I initially thought was the final step.

**Verification is part of deployment.**

If something goes wrong, the process should help us identify the problem, fix it, and safely redeploy.

---

## Group Exercise: Ship NYC Mobility

For our group exercise, we had to think of one simple change we could introduce to our NYC Mobility homework and map it through a CI/CD process.

Our group tried this:

```text
Modify notebook
      ↓
Push to GitHub
      ↓
CD deploys it
      ↓
Databricks
```

We didn't create a separate repository for this. We worked with our existing project and used a notebook change as the example.

The exercise helped connect the lecture to something we've actually been working on. We saw how even a small change to our homework could go through a controlled development → GitHub → deployment workflow.

---

## Orchestration

Another topic was orchestration.

The basic concepts were:

```text
Jobs
 ↓
Tasks
 ↓
Dependencies
```

A pipeline isn't always just one script running from beginning to end because different tasks may depend on each other.

For example:

```text
Ingest data
     ↓
Validate data
     ↓
Transform data
     ↓
Load Gold
     ↓
Run analytics
```

The orchestration layer needs to understand those dependencies so tasks happen in the correct order.

---

## Parameterization

Parameterization was another concept that I found useful.

The basic idea is:
> Instead of writing a separate piece of code for every value, make the value a parameter.

For example, instead of hard-coding every month:

```text
March
April
May
```

we can make the month a variable:

```python
process_month(month)
```

Then:

```python
process_month("2026-03")
process_month("2026-04")
process_month("2026-05")
```

The goal is reusable code.

### Hard-coding vs. Overengineering

Hard-coding or brute forcing can make code repetitive and fragile but the opposite extreme is also a problem.

Trying to make everything extremely generic can lead to **overengineering**.

So there's a balance:

```text
Hard-coded
     ↓
Too specific
     ↓
Fragile

      ↕
   Balance

      ↕

Overengineered
     ↓
Too complex
     ↓
Hard to maintain
```

The goal is:

> **Highly reusable and efficient code without unnecessary complexity.**

---

## Scheduling

Another thing that also became very clear during this session:

> **Failure is normal.**

Things will fail.

For example:

* APIs time out.
* Files arrive late.
* Schemas change.
* Code contains bugs.
* Source data is bad.

So a reliable pipeline is a pipeline that **fails safely and can recover** (not necessarily a pipeline that never fails).

---

## What does "safe" mean?

A failure is an _unsuccessful meeting of an operational condition_, according to Sir Myk.

For example:

```text
Expected:
File arrives by 8:00 AM

Actual:
File doesn't arrive

→ Failure
```

The important part is what happens next.

We shouldn't blindly rerun the pipeline.

Different failures need different responses.

```text
API timeout
    ↓
Retry / backoff

Missing file
    ↓
Wait / alert

Schema change
    ↓
Stop / investigate

Bad source data
    ↓
Quarantine / investigate

Code bug
    ↓
Fix code → test → redeploy
```

The response depends on the type of failure.

---

## Detect → Diagnose → Recover → Verify

A reliable system should be designed around the failure lifecycle:

```text
Failure
   ↓
Detect
   ↓
Diagnose
   ↓
Recover
   ↓
Verify
```

This is much better than:

```text
Failure
   ↓
Rerun everything
   ↓
Hope it works
```

The system needs enough observability and validation to tell us:

1. Something failed.
2. What failed.
3. Why it failed.
4. How to recover.
5. Whether the recovery actually worked.

---

## Incident vs. Problem

Another distinction from the session:

**Incident and problem are not the same thing.**

An incident is an event at a specific point in time.

For example:

> "The pipeline failed at 10:32 AM."

The underlying problem might be:

> "The API authentication token expired."

So:

```text
Incident
= What happened?

Problem
= Why did it happen?
```

Understanding this distinction matters because fixing the immediate incident _doesn't necessarily fix the underlying problem_.

---

## System = Sustainability

One of my biggest takeaways from this session is that production engineering is really about **sustainability**.

It's easy to think about a pipeline as:

```text
Input → Code → Output
```

But a production system is more like:

```text
Input
  ↓
Pipeline
  ↓
Output
  ↓
Monitoring
  ↓
Failure Detection
  ↓
Diagnosis
  ↓
Recovery
  ↓
Verification
  ↺
```

The pipeline needs to keep working even when:

* the source changes
* data is late
* APIs fail
* code changes
* developers change
* requirements change

That is what makes the system sustainable.

### Final takeaway

The biggest shift in my thinking from this lecture:

> **A pipeline isn't reliable because it doesn't fail. It's reliable because we know how it behaves when it does fail.**

CI/CD, orchestration, parameterization, scheduling, monitoring, and recovery are all pieces of the same bigger idea:

**We're not just writing code that works. We're building systems that can keep working.**
