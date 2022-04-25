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

JSDocs are amazing! While they make a big difference on obvious components (e.g. `Button`), they are amazing to clarify and document utility functions, hooks, complex components and such.

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

### 👉 Avoid adding `geek` or `{projectName}` prefix

When naming variables, it's a common practice to add `geek` or any other prefix related to the company or project to make that variable unique.

It's a great idea to avoid doing this, since everything inside a `geek` repo belongs to `geek` - so it's redundant.

❌ Bad

```tsx
export const colors: ThemeColors = {
  geekPrimary: {
    50: "#ffffff",
    100: "#F0F0FF",
    // ...
  },
};
```

✅ Good

```tsx
export const colors: ThemeColors = {
  primary: {
    50: "#ffffff",
    100: "#F0F0FF",
    // ...
  },
};
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

# :pushpin: Atomic Design Testing

- [Atoms](#atoms)
  - Snapshots
- [Molecules](#molecules)
  - Snapshots (mocked)
  - Integration tests
- [Organisms](#organisms)
  - Snapshots (sometimes mocked)
  - Integration tests
- [Utils, hooks, shared...](#utils-hooks-shared)
  - Unit tests

# Atoms

## 👉 Snapshot tests

> 💡 Snapshot tests are a very useful tool whenever you want to make sure your UI does not change unexpectedly.

Snapshot testing is a pretty controversial, some people love it and others hate it, and honestly I understand both. Among the main complaints about snapshot testing are:

1. Go against TDD
2. They are hard to maintain in large teams
3. They are general, we often just re-generate the snapshot without debugging the error

Are any of these points wrong? It depends.

If we look at a standard component structure divided by modules, the snapshot testing approach may not make sense, because we have large components like e.g. registration form, header, comments that need larger tests like integration tests in addition to unit tests

Now, if we look at atomic design, an atom should be a stylized html tag and just that, the smallest part of an organism without any kind of logic. In other words, we need to validate that it is just rendering the content and styles correctly, exactly what a snapshot test does.

### 👎 Without snapshot tests:

```typescript
it("should render the component correctly", () => {
  const { getByTestId } = render(<Button variant="outline">Teste</Button>);

  expect(getByTestId("button")).toBeInTheDocument();
  expect(getByTestId("button")).toHaveStyle("background-color: transparent");
});
```

### 👍 With snapshot tests:

```typescript
it("Renders correctly button props as solid", () => {
  const tree = renderer.create(<Button variant="solid">Teste</Button>).toJSON();

  expect(tree).toMatchSnapshot();
});
```

With snapshot we are testing all passed properties, and if any property changes, the test will inform us, leaving it up to the dev in action and the devs in code review to review this change to avoid future problems. Since we use a third-party library (**Chakra ui**) there is no need to test property rendering, as all components are already tested before they are released.

🤔 About TDD, the point that snap tests are done after the component has been developed is extremely valid, but again outside the context of atomic design, in my opinion. Generally, when we create a component in an atomic design framework, we start by creating a large component (Organism, which can be tested in many ways) and split it into small molecules and small atoms (which are already working in an organism context) and just need to check if their styles are working. From the moment an atom has logic that needs to be tested, something is not right.

[Back to atomic desing testing ⬆️](#pushpin-atomic-design-testing)

# Molecules

## 👉 Testing snapshots? Again? Yes, but different.

It may seem strange to use snapshot tests again on molecules if the molecules are made of atoms already tested with this tool, but the purpose here is different. On the atoms we want to test if the style of the components, classes and attributes are correct. On molecules, made up exclusively of atoms, we don't need to revalidate the styles, classes and attributes, we just need to know if the molecule is correctly calling its atoms, and if one is changed by removing or adding a new atom, the snapshot will indicate this change, this is why we use shallow rendering (or as we call it, mock snapshot)

### 👉 Normal rendering

```typescript
exports[`Renders correctly button props as outline 1`] = `
<button
  className="chakra-button css-7u2sko"
  data-testid="button"
  type="button"
>
  Teste
</button>
`;
```

### 👉 Shallow rendering

```typescript
exports[`MaskedField molecule  Renders correctly outline field props 1`] = `
<FormControl
  position="relative"
  width="100%"
>
  <FieldTitle
    isRequired={true}
    label="Qual a URL do seu perfil do LinkedIn?"
  />
  <InputGroup
    flexDirection="column"
  >
    <InputLeftElement
      cursor="default"
      height="40px"
    >
      <Icon
        as={[Function]}
        color="text.100"
        h={5}
        mx={4}
        w={5}
      />
    </InputLeftElement>
    <Input
      _focus={
        Object {
          "bgColor": "secondary.50",
          "borderColor": "secondary.50",
          "boxShadow": "none",
        }
      }
      _hover={
        Object {
          "borderColor": "secondary.50",
        }
      }
      as={[Function]}
      beforeMaskedValueChange={[Function]}
      bgColor="background.900"
      borderColor="background.900"
      borderRadius={4}
      color="text.100"
      data-testid="masked-input"
      fontSize="14px"
      fontWeight="400"
      formatChars={
        Object {
          "*": ".",
          "9": "[0-9]",
          "x": "[аa-zà-úÀ-ÚüÜA-Z0-9%-]",
          "z": "[A-Za-zà-úÀ-Ú]",
        }
      }
      height="40px"
      inputRef={null}
      mask="https://www.linkedin.com/in/********************************************************************************************"
      maskChar={null}
      paddingRight="12px"
      placeholder="https://www.linkedin.com/in/<seu_linkedin>"
      py={2}
      type="text"
      width="100%"
    />
  </InputGroup>
  <FormHelperText
    color="text.200"
    data-testid="help-text"
    fontSize="0.9rem"
    fontStyle="italic"
  >
    Digite a url do seu perfil
  </FormHelperText>
</FormControl>
`;
```

## 👉 Ok, why the integration test?

We know that the molecule is unchanged and is calling its atoms correctly, now we need to see if this set of atoms is actually working as we expect, literally to test its integration. With this we test the purpose of that component with jest, for example:

```typescript
// more code...

const field = getByTestId("masked-input");

fireEvent.focus(field);

expect(field).toHaveValue("https://www.linkedin.com/in/");

fireEvent.change(field, {
  target: { value: "https://www.linkedin.com/in/danisanc" },
});

expect(field).toHaveValue("https://www.linkedin.com/in/danisanc");

fireEvent.change(field, {
  target: { value: "www.linkedin.com/mwlite/in/" },
});

expect(field).toHaveValue("https://www.linkedin.com/in/");

fireEvent.emptied(field);

expect(field).toHaveValue("https://www.linkedin.com/in/");
```

[Back to atomic desing testing ⬆️](#pushpin-atomic-design-testing)

# Organisms

## 👉 Snapshots, but with a twist

For organisms we decided to keep using snapshots, because we saw in snapshots an easier way to get any layout break.
Doing only integration tests we won't cover the cases when a style prop like margin or padding, passed down to a child component, has changed.

So, we are keeping the snapshots, but we can't always use shallow snapshots here.
If we need a mocked provider or if we have to mock a mutation from Apollo for our form submit, for instance, the shallow render will break. In this case we will use a standard render, like this:

```typescript
const { container } = render(<RegisterStepLinkedin />);
expect(container).toMatchSnapshot();
```

The rule of thumb is always goes for a shallow render in organisms snapshots, if you can't do it, then use the standard way.

And yes, if we are using standard render, when a child component snapshot breaks, it will also break the organism snapshot. This is a drawback we are fine with.

**It's important to note that all the broken snapshots should be updated in the same PR.**

## 👉 Integration tests

If the organism doesn't have a form to submit, then we do the same way as we did with the molecules integration tests, but to be able to do the integration tests using React Testing Library and React Hook Form, we needed to make some changes:

1. We needed a `form` tag, so we updated our `FormContainer` component to a form

```typescript
export const FormContainer = ({
  children,
  ...props
}: PropsWithChildren): ReactElement => (
  <Box
    as='form'
    bgColor='background.700'
...
```

2. We needed the FormContainer to have the `onSubmit` function, for instance:
   `<FormContainer onSubmit={methods.handleSubmit(onSubmit)}>`

3. We needed the form submit button to have a `type='submit'`
4. In order to find the inputs using the react testing library we needed to add an `aria-label` to them. For the inputs the value of the `aria-label` should be the same as the input label and for the buttons it should be the same as the button text. Doing it this way we keep the aria-label accessibility purpose too.

With all of this checked we can create our integration tests.

**It's worth to remember that we should test every scenario in the form: warnings messages, errors message, field validation (we already have yup validation to tell us the cases we need to test!!)**

## 👉 Best practices for integration tests with React Hook Form

- Use `userEvent` to simulate user typing, clicks and etc
- To find the inputs use `screen.getByRole`
- To find other elements in the DOM (like errors messages) use `screen.getByText`
- To test the errors messages that should be displayed, if it's always the same message we can use `findAllByRole('alert')` e validate the number of errors expected, but if there are multiple error messages (different yup validations) for a field we should create a test for each one and validate de error text using `screen.getByText`.

```typescript
const alert = await screen.findAllByRole("alert");
expect(alert).toHaveLength(2);

const error = screen.getByText(/Insira uma URL válida/i);
expect(error).toBeInTheDocument();
```

In the example above we should have 2 alerts in the DOM one is a warning message with always the same text and the other one is the linkedin validation error for a invalid URL.
We could also have another validation error message when the linkedin field is empty, which we should check for in another test.

- To test the form submit we should use `toHaveBeenCalledWith` with the values we filled the inputs with.

```
 waitFor(() => {
      expect(mockMutate).toHaveBeenCalledWith({
        variables: {
          searchStatus: 'searching_for_a_job_right_now',
          linkedInUrl: 'https://www.linkedin.com/in/johndoe',
          acceptLinkedinImport: undefined,
        },
      });
    });

```

[Back to atomic desing testing ⬆️](#pushpin-atomic-design-testing)

# Utils, hooks, shared...

## Unit tests

We have no secrets around here, do we? Hooks, utils, or shared functions are usually simple functions with one or a few input values and one or a few output values, and all you need to do is unit test to see if the internal processing of that data is working as it should

I could give several examples of complex tests here, but I think that with the basics we already understand. In theory, hooks, utils, and shared functions are fractions of code that we extract and separate so they can be reused in other parts of the application without code redundancy, like this simple sum function:

```typescript
function sum(a, b) {
  return a + b;
}
```

Which can be tested with a simple:

```typescript
const result = sum(1, 2);
expect(result).to.equal(3);
```

Regardless of the complexity of the tests, the idea will always be the same, passing the expected values on the input should test the expected value on the output.

[Back to atomic desing testing ⬆️](#pushpin-atomic-design-testing)

[Back to top ⬆️](#pushpin-summary)

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

[Back to top ⬆️](#pushpin-summary)
