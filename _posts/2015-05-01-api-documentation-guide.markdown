---
layout: post
title: "API Documentation Guide"
short_title:  "API Documentation Guide"
date:   2014-11-01 23:08:50
sections:
    - 'Implementation Details'
<!-- redirect_from: -->
---

Getting **API documentation** done right is hard. Keeping it up to date is even harder. Yet it's a critical part of every microservice. API documentation is a **"face" of microservice** (read: dev team) because it's the first thing that people see, and better it be slick and fresh. Below are few tips and approaches how to achieve it.

## API Documentation goals

Perfect API documentation:

- is **easy to navigate** and understand
- **automatically generated**
- **integrated with tests**:
    + docs are used as expectation against endpoints
    + executed as part of test build (i.e. with rspec)
- should be **easy to modify**:
    + has support of partials
    + support of common parameters
- contains **examples**:
    + example body of the request and response (with values that make sense)
    + example `curl` or `httpie` commands to execute
- provides API access in a browser via API console

