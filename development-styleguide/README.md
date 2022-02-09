<p align="center">
  <img alt="logo" src="/docs/logo.png" width="200">
</p>

# 💻 The Geek Developer Style Guide

Welcome to our development style guide!

- The goal of this guide is to help our team to understand and follow our development style and best practices of the Geek Developer.
- This style guide is a living document that will be updated as we continue to develop our code.


`#letscode 😎👩‍💻👨‍💻`
# :pushpin: Summary

- [Introduction](#introduction)
- [Communication](#organization)
- [Code Style](#code-style)

## Introduction

- As we are growing as a team, focused on being a <b>reference</b> in software development, we need to be able to communicate sync and async in a way that is easy to understand for everyone. We use [Slack](https://slack.com/intl/pt-br/) as our sync communication tool and [GitHub](https://github.com/GeekHunter-Brasil) as our async communication tool.
- We also feel the need to establish a common language to share our knowledge and experience in the development process as much as a common way to report issues, bugs, performance issues, and improvements proposals in architecture or infrastructure. That language is [English](https://en.wikipedia.org/wiki/English_language).
- Talking about code, the traceability of the code is important to us, we need to be able to easily understand the reasons why a piece of code is there. See this reference about [Semantic Commits](https://gist.github.com/joshbuchea/6f47e86d2510bce28f8e7f42ae84c716).
- When dealing with a legacy codebase you may feel in need to refactor it or you may also feel in need to improve the codebase to make it more maintainable, 
and therefore, you will need to apply [Agile](https://blog.trello.com/microproductivity-break-tasks-into-smaller-steps) and [Solid](https://en.wikipedia.org/wiki/Single-responsibility_principle) principles to do so. 

[Back to top ⬆️](#pushpin-summary)


## Communication

- It's easier to keep the main language in English for async communication since our code is 100% English - that means less context switching and more efficiency. 
- Document/communicate in English helps us to better consume/collaborate with the tech community as a whole.
- Another reason to use English as official language for PRs/CRs is that it avoids unnecessary translations of db fields and/or terms.
- Whenever you feel the need to start a discussion or anything similar, prefer structured communications like Github Issues instead Slack.
- If you believe you need the acknowledgment of the whole chapter about something you have to say or do, on behalf of the chapter, consider opening an Issue.
- It will make far more easy to newcomers to reach this topics and issues that may, in sometime, be their issues.
- It also improves a lot our documentation about our own development as a reference software engineer team.

- Use the repositories Issues tab to create issues related to that project and further a task related to that issue and then a PR related to the task is a good cycle of improvement.

[Back to top ⬆️](#pushpin-summary)


## Code Style

- It's a good approach to do not change files not related to you PR. If some adjustment is needed, then another PR should be made with another meaningfull commits. This extends from the single responsibility principle.
- PRs should have one and only one objective. A PR can do either of these things, but not both:
    - Refactor existing code to format it
    - Add a new feature
    - Fix a bug
    - Improve performance
    - Etc
- In a small PR the reviewer has a bigger chance of noticing/pointing out changes in files/lines that are clearly out of scope.
- Read [this article](https://essenceofcode.com/2019/10/29/the-art-of-small-pull-requests/) for more info about the art of small pull requests

[Back to top ⬆️](#pushpin-summary)
