<p align="center">
  <img alt="logo" src="/docs/logo.png" width="200">
</p>

# 🎨 Frontend Style Guide

Welcome to our frontend style guide! Grab your cup of coffee (or tea) and read this with attention. ☕🍵

The goal of this guide is to help our team to understand and follow our code style and best practices, maintaining a pattern in all repositories.

`#letscode 😎👩‍💻👨‍💻`

# :pushpin: Summary

* [Introduction](#introduction)
* [Organization](#organization)
* [GraphQL](#graphql)
* [Code Style](#code-style)
* [Tests](#tests)

## Introduction

Internally, we use [React](https://reactjs.org/) with [NextJS](https://nextjs.org/). All projects use [TypeScript](https://www.typescriptlang.org/).

To communicate with our backend, we use [GraphQL](https://graphql.org/), along with [GraphQL Code Generator](https://www.graphql-code-generator.com/).

To setup our components using our Design System, we use [Chakra UI](https://chakra-ui.com/).

To test our application, we use [Jest](https://jestjs.io/pt-BR/), [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/) and [Cypress](https://www.cypress.io/).

Finally, to enforce code style in our repositories, we use [ESLint](https://eslint.org/) and [Prettier](https://prettier.io/).

It's a great idea to read the [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript) before continuing, since we use it as a base.

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

We use GraphQL to communicate with our backend. In our frontend apps, we also use [GraphQL Code Generator](https://www.graphql-code-generator.com/) to automatically generate React code based on our query files.

To do this, we organize all our query, mutations and subscriptions inside the `src/lib/graphql` folder. If needed, we can also create more folders to define contexts and properly organize our files:

```
📁 project/
└──📁 src/
   └──📁 lib/
      └──📁 graphql/
         └──📁 queries/
            └──📁 candidates/             # Queries related to candidates
            └──📁 companies/              # Queries related to companies
         └──📁 mutations/
            └──📁 ...
         └──📁 subscriptions/
            └──📁 ...
```

For specific query files, we usually use `export default` with a named query:

```typescript
import { gql } from '@apollo/client';

export default gql`
  query GetCandidate ($id: String!) {
    getCandidate (id: $id) {
      name
      email
    }
  }
`;
```

Using this pattern, we can use `graphql-codegen` to auto-generate code based on all files located in `src/lib/graphql`. This means that:

- ✅ We don't need to write any code related to consuming and handling requests;
- ✅ If anything changes in our backend (e.g. a field is removed), the frontend validation pipelines break (due to TypeScript errors);
- ✅ If we forget to query a field and render this field, the frontend validation pipelines break (due to TypeScript errors);

[Back to top ⬆️](#pushpin-summary)

## Code Style

Here we define code style patterns that should be followed when writing code.

It is a great idea to read the documents below, since we use them as a base:

- [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)
- [Prettier.io](https://prettier.io/docs/en/index.html)

### When possible, write JSDocs 📃

JSDocs are amazing! While they make a big difference on obvious components (e.g. `Button`), they are amazing to clarify and document utility functions, hooks, complex components and such.

Check out these guides:
- [JSDocs](https://jsdoc.app/)
- [Document your Javascript code with JSDoc](https://dev.to/paulasantamaria/document-your-javascript-code-with-jsdoc-2fbf)

❌ Bad
```tsx
const formatAge = (age: number): string => {
  // Implementation
}
```

✅ Good
```tsx
/**
 * Formats an age (number) to a string description.
 * @param age The person age.
 */
const formatAge = (age: number): string => {
  // Implementation
}
```

<br />

### Avoid using `React.FC`

Internally we avoid using `React.FC` due to some very rich discussions ([1](https://fettblog.eu/typescript-react-why-i-dont-use-react-fc/), [2](https://github.com/facebook/create-react-app/pull/8177), [3](https://dmitripavlutin.com/typescript-react-components/)).

❌ Bad
```tsx
const Button: React.FC<ButtonProps> = (props) => {
  // Implementation
};
```

✅ Good
```tsx
const Button = (props: ButtonProps): React.ReactElement => {
  // Implementation
};
```

When declaring a component that has `children`, use a helper type:

```tsx
const Button = (props: PropsWithChildren<ButtonProps>): React.ReactElement => {
  // Implementation
};
```

<br />

### Do not use values not present in the theme

Values that are defined in our theme come from our Design System. We should use these values, and should not input manual values in our components.

❌ Bad
```tsx
<Box backgroundColor="#9412DC" />
```

✅ Good
```tsx
<Box backgroundColor="primary.500" />
```

<br />

### Avoid adding `geek` or `{projectName}` prefix

When naming variables, it's a common practice to add `geek` or any other prefix related to the company or project to make that variable unique.

It's a great idea to avoid doing this, since everything inside a `geek` repo belongs to `geek` - so it's redundant.

❌ Bad
```tsx
export const colors: ThemeColors = {
  geekPrimary: {
    50: '#ffffff',
    100: '#F0F0FF',
    // ...
  },
};
```

✅ Good
```tsx
export const colors: ThemeColors = {
  primary: {
    50: '#ffffff',
    100: '#F0F0FF',
    // ...
  },
};
```

<br />

[Back to top ⬆️](#pushpin-summary)

## Tests

We use [Jest](https://jestjs.io/pt-BR/) and [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/) to write unit tests for our components.

Also, we use [Cypress](https://www.cypress.io/) to write integration tests covering important application flows.

### Make sure to test component logic

When a component has logic inside it (e.g. rendering something based on a condition), this should be tested with unit tests.

Example: Below, we should at least create a test passing a string and `undefined` to `title`.

```tsx
const Button = ({ title }: ButtonProps): React.ReactElement => (
  <button>{title ? title : 'Foobar'}</button>
);
```

### Make sure to test utility functions

When creating a new utility function, it's really important to test it. Testing utils is easy with unit tests, since they mainly deal with data or calling other functions.

```tsx
// formatName/index.ts
const formatName = (first: string, second: string): string => `${first} ${second}`;

// formatName/index.spec.ts
it('Correctly formats the name', () => {
  expect(formatName('Foo', 'Bar')).toBe('Foo Bar');
});
```


[Back to top ⬆️](#pushpin-summary)
