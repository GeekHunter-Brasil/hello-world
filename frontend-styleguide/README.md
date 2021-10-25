<p align="center">
  <img alt="logo" src="/docs/logo.png" width="200">
</p>

<h2 align="center">
  Frontend Style Guide
</h2>

Welcome to our frontend style guide!

Grab your cup of coffee (or tea) and read this with attention. ☕🍵

## Intro

The goal of this guide is to help our team to understand and follow our code style and best practices, maintaining a pattern in all repositories.

We also use the [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript) as a base, so it is **highly recommended** reading it before you continue.

`#letscode 😎👩‍💻👨‍💻`

# :pushpin: Summary

* [Common Tools and Dependencies](#common-tools-and-dependencies)
* [Organization](#organization)
* [GraphQL](#graphql)
* [Code Style](#code-style)
* [Tests](#tests)

## Common Tools and Dependencies

TODO

[Back to top ⬆️](#pushpin-summary)

## Organization

We use Atomic Design to organize our components internally.

It is a great idea to read these awesome docs on Atomic Design:

- [Atomic Design and ReactJS, by Danilo Woznica](https://danilowoz.com/blog/atomic-design-with-react)
- [Rethinking Atomic React, by Cheesecake Labs](https://cheesecakelabs.com/br/blog/rethinking-atomic-design-react-projects/)

In short, the file organization for the application should be something like:

```
📁 project/
└──📁 src/
   └──📁 components/
      └──📁 atoms/                        # All atom components
      └──📁 molecules/                    # All molecule components
      └──📁 organisms/                    # All organism components
      └──📁 templates/                    # All template components

   └──📁 lib/                             # Any code that does not belong to `components` or `pages`
      └──📁 @types/                       # Global type declarations
      └──📁 constants/                    # Application constants
      └──📁 generated/                    # Auto-generated code
      └──📁 graphql/                      # Graphql files (queries, mutations, subscriptions)
      └──📁 hooks/                        # Application hooks
      └──📁 schemas/                      # Validation schemas (from Yup or Zod)
      └──📁 settings/                     # Config files for our dependencies (e.g. ChakraUI application theme)
      └──📁 utils/                        # Utility functions

   └──📁 pages/                           # Pages directory, from NextJS
      └──📁 example-route/
         └──📝 index.tsx
      └──📁 example-route-with-params/
         └──📁 [id]
            └──📝 index.tsx
```

As for component organization, we usually define all related files (e.g. tests, stories) in the same component folder. Example:

```
📁 project/
└──📁 src/
   └──📁 components/
      └──📁 atoms/
         └──📁 button/
            └──📝 index.tsx
            └──📝 button.stories.tsx
            └──📝 button.spec.tsx
```

[Back to top ⬆️](#pushpin-summary)

## GraphQL

TODO

[Back to top ⬆️](#pushpin-summary)

## Code Style

TODO

[Back to top ⬆️](#pushpin-summary)

## Tests

TODO

[Back to top ⬆️](#pushpin-summary)
