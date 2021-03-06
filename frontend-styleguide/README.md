<p align="center">
  <img alt="logo" src="/docs/logo.png" width="200">
</p>

# 🎨 Frontend Style Guide

Welcome to our frontend style guide!

The goal of this guide is to help our team to understand and follow our code style and best practices, maintaining a pattern in all repositories.

`#letscode 😎👩‍💻👨‍💻`

# :pushpin: Summary

- [Introduction](#introduction)
- [Organization](#organization)
- [GraphQL](#graphql)
- [Code Style](#code-style)
- [Tests](#tests)
- [Translate](#translate)

## Introduction

Internally, we use [React](https://reactjs.org/) with [NextJS](https://nextjs.org/). All projects use [TypeScript](https://www.typescriptlang.org/).

To communicate with our backend, we use [GraphQL](https://graphql.org/), along with [GraphQL Code Generator](https://www.graphql-code-generator.com/).

To setup our components using our Design System, we use [Chakra UI](https://chakra-ui.com/).

To test our applications, we use [Jest](https://jestjs.io/pt-BR/) and [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/).

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
import { gql } from "@apollo/client";

export default gql`
  query GetCandidate($id: String!) {
    getCandidate(id: $id) {
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

### 👉 When possible, write JSDocs 📃

JSDocs are amazing! While they don't make a big difference on obvious components (e.g. `Button`), they are amazing to clarify and document utility functions, hooks, complex components and such.

Check out these guides:

- [JSDocs](https://jsdoc.app/)
- [Document your Javascript code with JSDoc](https://dev.to/paulasantamaria/document-your-javascript-code-with-jsdoc-2fbf)

❌ Bad

```tsx
const formatAge = (age: number): string => {
  // Implementation
};
```

✅ Good

```tsx
/**
 * Formats an age (number) to a string description.
 * @param age The person age.
 */
const formatAge = (age: number): string => {
  // Implementation
};
```

<br />

### 👉 Avoid using `React.FC`

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

### 👉 Avoid using `&&` on conditional renders

It's a common practice to use `&&` to conditionally renders elements inside React components. The inadverted use of this pattern can lead to some unexpected errors.

It's a great idea to use ternary operators instead. For further reference, you can check [this article by Kent C. Dodds](https://kentcdodds.com/blog/use-ternaries-rather-than-and-and-in-jsx)

❌ Bad

```tsx
<Box>
  {errorMessage && <Text>{errorMessage}</Text>}
</Box>
```

✅ Good

```tsx
<Box>
  {errorMessage ? <Text>{errorMessage}</Text> : null}
</Box>
```

<br />

### 👉 Do not use values not present in the theme

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

### 👉 Avoid redundant names

When naming variables, it's a common practice to overexplain variables in their names.

Try to keep it simple and do not explain things that are already obvious from context or typing. 

❌ Bad

```tsx
const frameworkList = ["React", "Angular", "Vue"]
const inputElement = Document.querySelector("#input")
const [filterState, setFilterState] = useState()
```

✅ Good


```tsx
const frameworks = ["React", "Angular", "Vue"]
const input = Document.querySelector("#input")
const [filter, setFilter] = useState()
```

<br />

### 👉 Avoid import React in jsx files

Since [React 17](https://reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html) we no longer need to import React in jsx files.

We can import only the components from react that we are using

❌ Bad

```tsx
import React from "react";

export const Component = (): React.ReactElement => {
  const [state, setState] = React.useState("");

  return <div>Component</div>;
};
```

✅ Good

```tsx
import { ReactElement, useState } from "react";

export const Component = (): ReactElement => {
  const [state, setState] = useState("");

  return <div>Component</div>;
};
```

<br />

### 👉 Avoid using `export default`

When exporting components, prefer to use the `export { Foo }` or `export class Foo` over `export default Foo` syntax.

Doing so, we prevent the same component from having different names and improve the discoverability of the file.

You can find other good reasons by adopting this syntax [here](https://basarat.gitbook.io/typescript/main-1/defaultisbad).

❌ Bad

```tsx
class Foo {
  ...
};

export default Foo;
```

✅ Good

```tsx
class Foo {
  ...
};

export { Foo };
```

<br />

### 👉 Static objects should be declared outside of the function block

This is the preferred way because otherwise the object will be recreated on every render, which is unnecessary.

❌ Bad

```tsx
export const MyComponent = (): React.ReactElement => {
  const hoverStyle = {
    color: "primary.50",
    bgColor: "errors.150",
    borderColor: "errors.150",
  };

  return <Link _hover={hoverStyle}>Blá</Link>;
};
```

✅ Good

```tsx
const hoverStyle = {
  color: "primary.50",
  bgColor: "errors.150",
  borderColor: "errors.150",
};

export const MyComponent = (): React.ReactElement => (
  <Link _hover={hoverStyle}>Blá</Link>
);
```

<br />

### 👉 Use `async` and `await` instead promises `then` and `catch`

When using promises or async functions, it's a common pratice use the `then` and the `catch` functions.

It's a great idea to use `async` and `await` instead this, since it's much shorter and helps reducing cognitive complexity.

Check out these articles related to promises and async/await ([1](https://betterprogramming.pub/javascript-best-practices-promises-45928fbfebe2), [2](https://medium.com/front-end-weekly/javascript-best-practices-promises-6c4f78a12c29)).
Check out these articles related to Apollo and GraphQL ([1](https://www.apollographql.com/blog/graphql/error-handling/full-stack-error-handling-with-graphql-apollo/), [2](https://www.apollographql.com/docs/react/data/error-handling/)).

❌ Bad

```tsx
const tryExecuteAsync = () => {
  exampleMethodAsync()
    .then((result) => {
      console.log(`executed successfully with result: ${result}.`);
    })
    .catch((err) => {
      console.log(`failed to execute with error: ${err}.`);
    });
};
```

✅ Good

```tsx
const tryExecuteAsync = async () => {
  try {
    const result = await exampleMethodAsync();
    console.log(`executed successfully with result: ${result}.`);
  } catch (err) {
    console.log(`failed to execute with error: ${err}.`);
  }
};
```

<br />

### 👉 Use `UPPER_CASE` and `camelCase` to declare constants

When declaring constants in different components or different folders, it is a good practice to standardize how we name them to improve the semantics of the code and regardless of where it is called, be easily recognized.

Use `UPPER_CASE` to declare constants in `/lib/constants` or any constant that is deeply immutable:

❌ Bad

```typescript
export const companiesLPHeaderLinks = [
  ...
];
```

✅ Good

```typescript
export const COMPANIES_LP_HEADERS_LINKS = [
  ...
];
```

Use `camelCase` to declare exported variables inside components, and `TitleCase` for types and interfaces:

❌ Bad

```typescript
[...]
import FieldTitle from 'components/atoms/form/fieldTitle';
import getErrorMessage from 'lib/utils/getErrorMessage';

// Interface and constant
import { variantsAvailable, FieldVariants } from './variants';

export interface FieldProps extends Omit<InputProps, 'h' | 'w'> {
variant?: variantsAvailable;
label?: string;
[...]
```

✅ Good

```typescript
[...]
import FieldTitle from 'components/atoms/form/fieldTitle';
import getErrorMessage from 'lib/utils/getErrorMessage';

// Interface and constant
import { VariantsAvailable, fieldVariants } from './variants';

export interface FieldProps extends Omit<InputProps, 'h' | 'w'> {
variant?: VariantsAvailable;
label?: string;
[...]
```

<br />

### 👉 Use theme tokens instead of hardcoded values

When styling, make sure to use the theme tokens provided by Chakra UI to prevent an unorganized code. You can check the default theme tokens [here](https://chakra-ui.com/docs/theming/theme).

For a quick example, you should use `'md'` or `{10}` instead of `"135px"` or `"0.5rem"`. Also, note: Use curly braces for numeric constants instead single quotes.

❌ Bad

```tsx
export const MyComponent = (): React.ReactElement => {
  return (
    <Flex>
      <Box mb="1rem">
        <Heading p="0.5rem" fontSize="1rem">
          Title
        </Heading>
      </Box>
      <Box>
        <Text fontSize="0.75rem">SubTitle</Text>
      </Box>
    </Flex>
  );
};
```

✅ Good

```tsx
export const MyComponent = (): React.ReactElement => {
  return (
    <Flex>
      <Box mb={4}>
        <Heading p={2} fontSize="md">
          Title
        </Heading>
      </Box>
      <Box>
        <Text fontSize="xs">SubTitle</Text>
      </Box>
    </Flex>
  );
};
```

<br />

### 👉 Use array syntax to handle responsive style props

Array syntax is the [recommended method for responsive style props in chakra-ui](https://chakra-ui.com/docs/styled-system/features/responsive-styles#the-array-syntax)

❌ Bad

```tsx
export const MyComponent = (): React.ReactElement => {
  return (
    <Box mb={{ sm: "1rem", md: "2rem", lg: "3rem"}}>
      <Heading fontSize={{ sm: "sm", md: "md", lg: "lg" }}>Title</Heading>
    </Box>
  );
};
```

✅ Good

```tsx
export const MyComponent = (): React.ReactElement => {
  return (
    <Box mb={["1rem", "2rem", "3rem"]}>
      <Heading fontSize={["sm", "md", "lg"]}>Title</Heading>
    </Box>
  );
};


### 👉 Static assets should be referenced by using `import`

When using static assets it should be referenced in the code by using `import` instead of reference by `string`.

This method makes the code more robust because ESLint will validate it.

❌ Bad

```tsx
{ src: '/companies/zup-color-new.png', alt: 'Zup', w: [85], h: [41] },
```

✅ Good

```tsx
import Carousel1 from '@public/companies/carousel1.webp';

...
panels: [
      {
        title: '📝 01. Cadastre-se',
        panelData: {
          imgSrc: Carousel1,
          imgAlt: 'Homem em reunião com CS da geekhunter',
...
```

<br />

[Back to top ⬆️](#pushpin-summary)

## Tests

We use [Jest](https://jestjs.io/pt-BR/) and [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/) to write unit tests for our components.

### 👉 Make sure to test component logic

When a component has logic inside it (e.g. rendering something based on a condition), this should be tested with unit tests.

Example: Below, we should at least create a test passing a string and `undefined` to `title`.

```tsx
const Button = ({ title }: ButtonProps): React.ReactElement => (
  <button>{title ? title : "Foobar"}</button>
);
```

### 👉 Make sure to test utility functions

When creating a new utility function, it's really important to test it. Testing utils is easy with unit tests, since they mainly deal with data or calling other functions.

```tsx
// formatName/index.ts
const formatName = (first: string, second: string): string =>
  `${first} ${second}`;

// formatName/index.spec.ts
it("Correctly formats the name", () => {
  expect(formatName("Foo", "Bar")).toBe("Foo Bar");
});
```

[Back to top ⬆️](#pushpin-summary)

### 👉 Avoid using `data-testid` to query elements

It's a common practice to define an attribute called `data-testid` and use it to query elements in component tests. Because `data-testid` is an arbitrary attribute, it says nothing about the element nor about its context. 

It's a great idea to query elements with more semantic functions, following the priority list defined on [React Testing Library documentation](https://testing-library.com/docs/queries/about/#priority).

❌ Bad

```tsx
const Input = getByTestId("submit-button")
const Input = getByTestId("candidates-heading")
const Input = getByTestId("login-link")

const Input = getByTestId("cpf-input")
const Input = getByTestId("name-input")
const Input = getByTestId("blog-paragraph")
```

✅ Good

```tsx
const Input = getByRole("button", { name: /enviar/i })
const Input = getByRole("heading", { name: /candidatos disponíveis/i })
const Input = getByRole("link", { name: /login/i })

const Input = getByLabelText(/cpf/i)
const Input = getByPlaceholderText(/ex: josé de araújo/i)
const Input = getByText(/loren ipsun loren ipsun loren ipsun/i)

```

<br />

## Translate

### 👉 Use I18n to translate

Our plataform has support for internationalization (i18n). We use [Next.Js](https://nextjs.org/docs/advanced-features/i18n-routing) built-in i18n and [next-translate](https://github.com/vinissimus/next-translate) to support rendering tokens on pages.

For each page created, it is necessary to update the pages key in the i18n.js file `i18n.js`:

```js
module.exports = {
  locales: ["pt-BR", "en"],
  defaultLocale: "pt-BR",
  loadLocaleFrom: (lang, ns) =>
    import(`./src/pages/locales/${lang}/${ns}.yaml`).then((m) => m.default),
  pages: {
    "/page_1": ["page_1"],
    "/page_2": ["page_2"],
  },
};
```

### 👉 Directory Locales structure

When organizing locale files you must insert each file into the translation folder grouped by pages, thus not creating large files. With this structure, we obtain a better organization.

```
📁 locales/
└──📁 pt-BR/
   └──📝 page_1.yaml
   └──📝 page_2.yaml
└──📁 en/
   └──📝 page_1.yaml
   └──📝 page_2.yaml
```

Each file inside contains its keys at the same level, there is no need to chain keys.

ex: `locales/pt-BR/page_1.yaml`.

```yml
example: Olá mundo
```

ex: `locales/en/page_1.yaml`.

```yml
example: Hello world
```

### 👉 Translate page

Use the translations on the page:

```tsx
import useTranslation from "next-translate/useTranslation";

export const Page = (): React.ReactElement => {
  const { t } = useTranslation("page_1");

  return <div>{t("example")}</div>;
};
```

```
  ⚠️ If the key is not found, next-translate prints the key`s names instead of the content.
  In the previous example, show example instead of Hello World.
```

To be able to load the translations it is necessary to compile the translations using the following command:

```
  yarn formatter-generator
```

[Back to top ⬆️](#pushpin-summary)
