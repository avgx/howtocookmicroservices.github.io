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
    + docs are used as expectation against endpoints. So for example if you update endpoint's template and forgot to update documentation your test suite will fail as your docs wouldn't be up to date
    + executed as part of test build (i.e. with rspec)
- should be **easy to modify**:
    + has support of partials
    + support of common parameters
- contains **examples**:
    + example body of the request and response (with values that make sense)
    + example `curl` or `httpie` commands to execute
- provides API access in a browser via API console

# Rails specific tools
We go over few tools and approaches we've tried, and cover pros and cons of each and come up with recommendation in the end. Of course no perfect solution exists for this problem, but there are different trade-offs here and there that we're going to discover and talk thru, so that you can find best tool to solve your problem.

[ ] fdoc
[ ] apipie
[ ] API Blueprint
[ ] pacto

### square/fdoc

[square/fdoc on github](https://github.com/square/fdoc)

Tool built by Square. You outline documentation in YAML language in a specific directory, where file path is the "url" and request/response bodies are inside. 

It's **good for** documenting **simple API** endpoints. But when your service gets any complexity, it becomes obvious that fdoc has **lack of support for partials** which makes it ridiculously hard to maintain endpoints that render same resource. For example when you have to add new attribute, you'd need to update documentation for every endpoint, rather then in just one place (if it had support for partials). On the bright side, **it supports automated testing** of endpoints **as part of your rspec suite**. So it takes care of the situations when developer updates view, but doesn't update documentation by failing the build.

##### PROS
- clean and simple approach
- generated documentation looks good and easy to navigate
- tightly integrated with rspec, executes and validates documentation as part of rspec suite

##### CONS
- no support of partials

##### OTHER
- no API console
- documentation is written in YML files

