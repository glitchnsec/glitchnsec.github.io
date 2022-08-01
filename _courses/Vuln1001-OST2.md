---
layout: course
code: Vulns1001
title: C-Family Software Implementation Vulnerabilities
institute:
    - name: OpenSecurityTraining
      url: "https://ost2.fyi/"
csite: "https://p.ost2.fyi/courses/course-v1:OpenSecurityTraining2+Vulns1001_C-derived+2022_v1/about"
date: 2022-05-18 00:00:00 -0600
collaborators:
    - name: Xeno Kovah
      url: "https://twitter.com/xenokovah?lang=en"
    - name: Kc Udonsi
      url: "https://glitchnsec.github.io"
summary: Buffer overflows, Out of Bound Writes, Integer Overflows and Underflows, Signed sanity checks, Integer truncation and Signed extension as seen in real world software in recent times
target: Any, developers, code auditors, vulnerability hunters
delivery:
    - format: Pre-Recorded
      subscription: Free
      enrollment: "https://p.ost2.fyi/courses/course-v1:OpenSecurityTraining2+Vulns1001_C-derived+2022_v1/course/"
    - format: Virtual/In-Person
      subscription: Custom Quote
      enrollment: 
---

### Description

This course is co-taught with [Xeno Kovah](https://twitter.com/xenokovah?lang=en). This is a dual-audience class. It targets developers who want to learn to write secure- or recognize unsecure- code. It's suitable for aspiring code auditors and freelance vulnerability hunters.

This class is structured in 5 main topic areas, corresponding to the vulnerability types namely:
- (linear) stack buffer overflows
- (linear) heap buffer overflows
- (non-linear) out-of-bound writes
- integer overflows/underflows, and "other integer issues" (signed sanity checks, integer truncation, and sign extension.)

For each topic area, we explain at least 6 real vulnerabilities.

Additionally, for at least one of those vulnerabilities we explain exploitation opportunities. Students will understand that exploitation engineering is just a typical engineering discipline, akin to a specialized form of software engineering.

At the end of each topic area, we cover prevention, detection, and mitigation opportunities corresponding to the vulnerability types
