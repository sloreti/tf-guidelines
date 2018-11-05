# tf-guidelines

> Programming quotes taken from [twitter.com/codewisdom](https://twitter.com/codewisdom).

The purpose of this repository is to lay out explicit guidelines for starting new projects and coding with *maintainability* and *extensibility* in mind. It contains rules that linters like ESLint or PEP8 will not recognize, but we as humans need to be aware of. Coding is designing a system and good design prevents errors.

- [tf-guidelines](#tf-guidelines)
  - [Starting new projects](#starting-new-projects)
  - [Working with branches](#working-with-branches)
  - [Programing recommendations](#programing-recommendations)
    - [Make use of the built-in features](#make-use-of-the-built-in-features)
    - [Copypaste versus abstraction](#copypaste-versus-abstraction)
    - [Object properties](#object-properties)
    - [Flat control flow](#flat-control-flow)
    - [Code should be beautiful](#code-should-be-beautiful)
    - [Middleware handling](#middleware-handling)
    - [styled-components usage](#styled-components-usage)
    - [styled-components nesting](#styled-components-nesting)
    - [styled-components prefixing](#styled-components-prefixing)
    - [Pass the props](#pass-the-props)
    - [Composable elements](#composable-elements)
    - [Functional component vs class component](#functional-component-vs-class-component)
    - [Declarative programing in React](#declarative-programing-in-react)

---

## Starting new projects

> "Take care of your tools and they will take care of you."

When creating new projects complete the following checklist before starting to code the actual requirements and sharing the codebase with the team:

- [ ] Follow the naming conventions.
  - GOOD: Platform repositories uses `tf-[name]` lowercase, so the new repository will be named `tf-cms`. <!--- TODO: Change tf to foundry, or whatever we decide on -->
  - BAD: Break the convention with `TFCMS`, `tfcms`, or similar.
- [ ] Create `.gitignore` and include the defaults for all operating systems (MacOS|Windows|Linux).
  - GOOD: Use https://www.gitignore.io/api/linux,macos,windows
- [ ] Add the platform specific ignore blobs to `.gitignore` (VS Code, Node.js, Python, Go, etc).
  - GOOD: Use https://www.gitignore.io
- [ ] **Set up the testing suite with at least one passing test!** At least one is needed to serve as reference for future development.
- [ ] Set up a code linter & formatter. This will help make code consistent throughout development.
  - GOOD: Use [tokenfoundry/eslint-config](https://github.com/tokenfoundry/eslint-config).
- [ ] If you need to expose a port be sure to not collide with the port of any existing TF projects.
- [ ] Set the metadata information (`package.json`) version to `0.0.0`.
- [ ] Create a `README.md` with at least the following information:
  - [ ] Purpose of the repository and how it relates to our other repositories.
  - [ ] Requirements like: Node.js >=8, Yarn, Docker 1.8, etc.
  - [ ] `bash`/`shell` commands to clone the repository.
  - [ ] Obviously the markdown output must be pretty and formatted, but please also use indentation and breaklines to have beautiful markdown source code as well.
  - [ ] A `.env` file is okay and encouraged (Be sure to add it to `.gitignore`).
    - GOOD: Use `dotenv` packages for Node.js, find similar for other languages.
  - [ ] For each environment (_development_, _production_, _testing_, _ci_):
    - [ ] Include the necessary environment variables in the `README.md`. If it is safe for the variable to be shared, inline it directly in the `README.md`. For private variables (passwords, keys) leave it empty and include a note stating where/who to ask for that value.
    - [ ] List the necessary commands to install and run the project.
    - [ ] Include where to access the running application or how to see its output.
    - [ ] Describe the common errors that may appear and how to fix them.


  ````md
  # tf-cms

  Manage content for [tf-frontend](https://github.com/tokenfoundry/tf-frontend) that can't be stored in a SQL database.

  ## Requirements

  * [Node.js](http://nodejs.org/) >=8
  * [Yarn](https://yarnpkg.com/) >=1.7

  ## Getting started

  ```sh
  git clone https://github.com/tokenfoundry/tf-cms
  ```

  ### Environment variables

  Create a `.env` file with the following:

  ```
  PORT=9000
  LANG=EN

  # Get yours at https://coolproject.com/dev or ask the team
  SERVICE_API_KEY=xxx-xxx-xxx-xxx
  ```

  ## Development

  ```sh
  yarn run dev
  ```

  Visit [`http://localhost:9000`](http://localhost:9000).

  ## Linting and formatting

  ```sh
  yarn run lint --fix
  ```

  ## Testing

  ```sh
  yarn test
  ```

  ## Production

  ...
  ````

---

## Working with branches

If the project is plain text and doesn't require team collaboration (like this repo), keep everything in `master` and create PRs as needed.

Otherwise, if the project is large and involves many people, the branching should be as follows:

- `master` branch: the current production build
- `develop` branch: the current staging build. This version should be considered stable.
- `feature/[name]` branches: work in progress (WIP) code that will eventually get **squashed and merged** into `develop`.
- `feature/[name]-[subname]` branches: for more complex features, a subset of the work required for the main feature branch. This help the team collaborate, track, and review current progress and identify problems before they arise.
- `chore/[name]` branches: work that is refactoring or related to the health of the codebase.
- `hotfix/[name]` branches: a correction to a detected bug.

It is important to mention that it is good to have an open PR even before the branch is done because it allows the team to review the code and detect problems _a priori_. <!--- TODO: Question this -->

---

## Programming recommendations

> "The purpose of software engineering is to control complexity, not to create it."

### Make use of built-in features

> "Any program is only as good as it is useful." - Linus Torvalds

It is common to try to write code quickly with your existing knowledge of the problem.

Before writing any code, ask yourself _"Is what I am going to write common to any software or language? Has someone before me likely encountered this same issue?"_, If the answer to either is yes, someone has probably already created a solution.

Always research what tools and libraries are already out there and you will end up writing fewer and better lines of code:

```js
const registrations = [];

// BAD, six lines of code. Fewer will do
const isApproved = true;
for (var i = 0; i < registrations.length; i++) {
  if (registrations["approved"] === false) { // please use `!registrations["approved"]`
    isApproved = false;
  }
}

// BAD, works but not very semantic and hard to understand
const isApproved = registrations.findIndex(r => !r["approved"]) > -1;

// GOOD, awesome! clear and semantic; doesn't require comments
const approved = registrations.every(r => r["approved"]);
```

### Copypaste versus abstraction

> "No abstraction is better than wrong abstraction."

Sometimes it may seem necessary to write a _class_ or abstraction when dealing with a repetitive task. This may lead to undue complexity, and risks introducing incorrect abstractions, so in some cases it is better just to repeat code or function calls.

Also it can prevent someone reading your code from having to jump from function to function. Try to keep everything close to where it is needed.

```js
const keyBy = require("lodash/keyBy");

// BAD: We don't need abstraction for everything
const createIndexUsersBy = (users, id, name, insensitive) => {
  let result = null;
  if (insensitive) {
    users = users.map(u => u.toLowerCase());
  }
  if (id || name) {
    result = id ? keyBy(users, id) : keyBy(users, name)
  }
  // ...
  return result;
}
const usersById = createIndexUsersBy(users, "", null, false);
const usersByName = createIndexUsersBy(users, null, "name", false);

// GOOD: It's clear and specific operations are handled by the developer
const usersById = keyBy(users, "id");
const usersByName = keyBy(users.map(u => ... })), "name");
```

### Object properties in Javascript

> "Explicit is better than implicit. Simple is better than complex, ..." -The Zen of Python

In Javascript you have three ways of accessing object properties:

1. `const { user } = this.props;`
    - Use this when you are accessing 1-5 variables of the object. Try to avoid renaming variables.
2. `const user = this.props.user;`
3. `const user = this.props["user"];`
    - Use this syntax when dealing with _external data_, as if it were a JSON object (it likely came from a JSON object, so treat it as such).

**Using `["brackets"]` for external data helps the developer understand that we are dealing with data we don't have control over**. Also it's clear that it's a JSON object and not an internal Javascript property or value.

```js
// BAD. Many lines of code and many variables are renamed.
function something(user, project) {
  const {
    name: projectName,
    ...
  } = project;
  const {
    name: userName,
    last_name: lastName,
    age,
    ...
  } = user; // is `user` a Javascript internal object or external data?
  return {
    naming: userName + " " + lastName,
    project: projectName,
    age,
    ... // many other stuff
  }
}

// GOOD
function something(user, project) {
  return {
    naming: [user["name"], user["last_name"]].filter(Boolean).join(" "),
    project: project["name"],
    age: Number(user["age"]), // Never trust external data.
    ... // many other stuff
  }
}

// GOOD
class App extends Component {
  static defaultProps = {
    user: PropTypes.object.isRequired,
  };
  render() {
    // We know `user` is known because it's a required prop.
    const { user, project, ...props } = this.props;
    return (
      <div>
        // we don't known `name`, it's external so use brackets.
        <h1>{user["name"]}</h1>
      </div>
    )
  }
}

```

### Flat control flow

> "A good programmer is someone who always looks both ways before crossing a one-way street."

Avoid temporal values, deep conditions and hard-to-follow escapes.

```js
// BAD, super-bad, impossible to follow.
pass = true;
if (user) {
  pass = true;
  if (user["data"]) {
    if (user["data"]["key"] !== "0x") {
      return "FALSE"; // BAD! make returns consistent
    }
  } else if (..) {
    pass = false;
  } else {
    return pass;
  }
  return true;
}
if (pass || !user || user && user["data"] || ...) {
  if (!pass) {
    ...
  }
  ...
}

// GOOD, clear control flow and adds _firewalls_ so that the "happy-path" return value is at the end.
if (!user) {
  return false;
} else if (!user["data"]) {
  return false;
} else if (!user["data"]["key"] || user["data"]["key"] !== "0x") {
  return false;
} else {
  return true; // happy return always at the end.
}
```

### Code should be beautiful

> "Read other people's code and 'steal' the good parts. Even if you try to do things exactly like someone else, you'll fail, but in doing so create something that's truly your own."

If you are feeling that you are writing a lot of code, it's probably because you are doing something wrong. Here some tips:

- Avoid temporal or auxiliary variables.
- Build mental maps or representations easy for humans.
- Read the language documentation.
- Prefer functional programming styles to imperative.
- Ask for help or recommendations.

<!--- TODO: This advice doesn't feel actionable to me -->

And finally, don't copy & paste code if you don't understand it first.

```js
// GOOD, easy to read and performant.
const [
  { data: project },
  { data: registrations },
  { data: questionnaires },
] = await Promise.all([
  await platform.GET(`/api/projects/${token["project"]}`),
  await platform.GET(`/api/token_sales/${sale["id"]}/sale_registrations`),
  await platform.GET(`/api/token_sales/${sale["id"]}/questionnaires`),
]);

return ctx.render({
  props: {
    registrations: registrations["results"],
    // ...
  }
})
```

### Middleware handling

TODO

### styled-components usage

> “Code is like humor. When you have to explain it, it’s bad.” – Cory House

When a prop changes many styles, group them with `css` instead of using complex ternary operations.

```js
import styled, { css } from "styled-components";

// BAD, complex to understand
const CommentBox = styled.div`
  border-color: white;
  color: ${props => props.locked ? "grey" : props.highlight ? "yellow" : "white"};
  text-transform: ${props => props.highlight ? "uppercase" : undefined};
  background-color: ${props => props.highlight ? "white" : "inherit"};
`;

// BAD, use the interpolation to add css `values;`, not css property `key: value;`
const CommentBox = styled.div`
  border-color: white;
  ${props => props.locked && `color: red;`}; // BAD
`;

// GOOD
const CommentBox = styled.div`
  border-color: white;
  color: ${props => props.locked ? "grey" : "white"};
  ${props => props.highlight && css`
    color: ${props => props.locked ? "grey" : "yellow"};
    text-transform: uppercase;
    background-color: white;
  `};
`;

// GOOD if the styled-component gets bigger.
const CommentBox = styled.div`
  border-color: white;
  ...
  ${props => props.highlight && props.locked && css`
    color: grey;
    ...
  `};
  ${props => props.highlight && !props.locked && css`
    color: yellow;
    ...
  `};
`;
```

### styled-components nesting

Avoid unspecific selectors.

```js
import styled from "styled-components";

const Panel = styled.div`
  ...
`;

const MyForm = styled.form`
  ...
`;

// BAD
const SignUpPanel = styled(Panel)`
  color: red;
  form {
    color: white;
  }
`;

// GOOD
const SignUpPanel = styled(Panel)`
  color: red;
  ${MyForm} {
    color: white;
  }
`;
```

### styled-components prefixing

Don't. `styled-components` handles vendor prefixing automatically. See: [styled-components.com/docs/basics#getting-started](https://www.styled-components.com/docs/basics#getting-started).

### Pass the props

> "Make it correct, make it clear, make it concise, make it fast. In that order." – Wes Dyer

When defining a component, **always** pass the `...props`. Otherwise this can be frustrating to other dev trying to extend the component and realizing that nothing happens and he/she must spend time debugging your mistake.

<!--- TODO: Can we majorly simplyify the accompanying examples? And this is a good practice because it allows us to extend the styles with styled-components? Still unclear on this one. -->

```js
import React, { Component } from "react";
import styled from "styled-components";

// Will omit prop-types in this example.

const AvatarCircle = styled.div`
  ...
`;
const AvatarImage = styled.img`
  ...
`;

// BAD, remember
const Avatar = ({ user }) => (
  <AvatarCircle>
    <AvatarImage src={user["profile_image_url"]} />
  </AvatarCircle>
);
// THIS WILL NOT STYLE THE COMPONENT!
const SquaredAvatar = style(Avatar)`
  ...
`;
// Bad usage example
class MyComponent extends Component {
  render() {
    // BAD, you are not passing the ...props
    const { user } = this.props;
    return (
      <div>
        // Props `style` & `className` will not working (neither styled-components)
        // ALSO BAD, do you really need to pass the whole `user` object as prop?
        <Avatar user={user} style={} className="something" />
        <SquaredAvatar user={user} style={} className="something-2" />
      </div>
    )
  }
}

// BAD-ish, see next section.
const Avatar = ({ image, ...props }) => (
  <AvatarCircle {...props}>
    <AvatarImage src={image} />
  </AvatarCircle>
);
// Will work, but you are only styling the outer `div`.
const SquaredAvatar = style(Avatar)`
  ...
`;
// Usage example
class MyComponent extends Component {
  render() {
    const { user, ...props } = this.props;
    return (
      <div {...props}>
        <Avatar image={user["profile_image_url"]} />
        <SquaredAvatar image={user["profile_image_url"]} />
      </div>
    )
  }
}

/**
 * CAN WE IMPROVE THIS? YES! Read "Composable elements" section below
 */
```

### Composable elements

> Read the [Pass the props](#pass-the-props) section before continuing with this section.

Want to create a sharable and reusable component? Do not create a `React.Component`, `React.PureComponent`, or `const Component = () => < />` component. **Create only styled-components**. Otherwise it is very easy to end over-propping your component.

**Expose its parts and let the developer compose and override the component part styles.**

> Example: I want to be able to style the inner `img` component and the outer `div` and also be able to add a child component.

```js
// BAD, and it's not a joke. It's a 200 LOC component.
const Avatar = ({
    image,
    innerClassname,
    innerStyle,
    innerProps,
    outerProps,
    width,
    height,
    margin,
    noPadding,
    isCircle,
    renderChildren,
    children,
    renderChildrenFirst,
    ...props
  }) => {
    let child = [];
    if (renderChildrenFirst) {
      if (renderChildren) {
        child.push(renderChildren());
      }
      if (children) {
        child.push(children);
      }
    } else {
      if (children) {
        child.push(children);
      }
      if (renderChildren) {
        child.push(renderChildren());
      }
    }
    const OuterComponent = isCircle ? AvatarCircle : styled(AvatarSquare)`
      ...
    `;
    // ...
    return (
      <OuterComponent noPadding width={width} height={isCircle ? width : height} {...outerProps} {...props}>
        <AvatarImage isSquare={!isCircle} noPadding className={innerClassname} style={innerStyle} src={image} {...innerProps} />
      </OuterComponent>
    );
};

// PERFECT, don't create nested Avatar component. Just it's parts
const Avatar = styled.div`
  ...
`;
Avatar.Image = styled.img`
  ...
`;
// PERFECT usage example
class MyComponent extends Component {
  render() {
    const { user, ...props } = this.props;
    return (
      <div {...props}>
        <Avatar>
          <Avatar.Image src={user["profile_image_url"]} />
        </Avatar>
      </div>
    )
  }
}

// PERFECT extending the component without over-propping
const AvatarSquare = styled(Avatar)`
  ...
`;
const AvatarImageSquare = styled(Avatar.Image)`
  ...
`;
// PERFECT Usage example
class MyComponent extends Component {
  render() {
    const { user, ...props } = this.props;
    return (
      <div {...props}>
        <AvatarSquare>
          <AvatarImageSquare src={user["profile_image_url"]} />
        </AvatarSquare>
      </div>
    )
  }
}
```

### Functional component vs class component

Our style guide prefers functional components over class based components, but you are free to use class based components when needed.

```js
// BAD, you are externalizing logic that corresponds to the component.
const getUserIsLogged = (user) => {
  return ...
}
const renderProfile = (user) => {
  return ...
}
const MyComponent = ({ user, ...props }) => {
  const isLogged = getUserIsLogged(user);
  const profile = renderProfile(user);
  return (
    ...
  );
}

// GOOD-ish, but please read the `Declarative programming in React` section.
class MyComponent extends Component {
  static isUserLogged = user => {
    return ...;
  };

  renderProfile(user) {
    //
  }

  render() {
    const isPrime = isNumberPrime(42);
    return ...;
  }
}

// GOOD, it's a Javascript or global thing and not related to the component.
const isNumberPrime = number => {};
```

### Declarative programming in React

Avoid imperative programming in React. Props and methods should describe behavior and should not force actions.

```js
import React, { Component } from "react";
import styled from "styled-components";

// BAD, you are forcing a color
const Button = styled.button`
  color: ${props => props.color || "grey"};
  :hover {
    color: ${props => props.colorOnHover || "grey"};
  }
`;

// GOOD, you are describing behavior.
const Button = styled.button`
  color: ${props => props.disabled ? "grey" : "blue"};
  :hover {
    color: ${props => props.disabled ? "grey" : "white"};
  }
`;

// ------------------------------------------------------------------

// BAD, getView is imperative
class App extends Component {
  state = {
    loading: false,
  };
  getView = () => {
    if (this.state.loading) {
      return <Loading />;
    } else {
      return <MyComponent />;
    }
  };
  render() {
    return (
      <div>
        {this.getView()}
      </div>
    )
  }
}

// BAD-ish, declarative but not enough
class App extends Component {
  state = {
    loading: false,
  };
  renderView = () => {
    if (this.state.loading) {
      return <Loading />;
    } else {
      return <MyComponent />;
    }
  }
  render() {
    return (
      <div>
        {this.renderView()}
      </div>
    )
  }
}

// GOOD-ish, declarative and kind-of-functional method (do not rely on state)
class App extends Component {
  state = {
    loading: false,
  };
  renderView = (isLoading = false) => {
    if (isLoading) {
      return <Loading />;
    } else {
      return <MyComponent />;
    }
  }
  render() {
    return (
      <div {...this.props}> // this good boi did not forget about props.
        {this.renderView(this.state.loading)}
      </div>
    )
  }
}

// PLEASE NO, don't create variables for components, makes the render method hard to read.
class App extends Component {
  state = {
    loading: false,
  };
  render() {
    const componentContent = this.state.loading ?  <Loading /> : <MyComponent />;

    // ...

    return (
      <div {...this.props}> // this good boi did not forget about props.
        {componentContent}
      </div>
    )
  }
}

// GOOD & AWESOME!, you understand the component just by looking the render method.
class App extends Component {
  state = {
    loading: false,
  };
  render() {
    return (
      <div {...this.props}> // this good boi did not forget about props.
        {this.state.loading ? (
          <Loading />
        ) : (
          <MyComponent />
        )}
      </div>
    )
  }
}


// PERFECT, Are you going to re-use the `MyComponent`? Probably not. Just inline it.
class App extends Component {
  state = {
    loading: false,
  };
  render() {
    // It's not bad having a big render method.
    // Remember HTML files? Those were big.
    // But you understand the big picture reading line-by-line without need to move your attention to methods.
    return (
      <div {...this.props}> // this good boi do not forget about props.
        {this.state.loading ? (
          <Loading />
        ) : (
          <Container>
            <Title>
              ...
            </Title>
            <Body>
              <Section>
                ...
              </Section>
              <Section>
                ...
              </Section>
            </Body>
          </Container>
        )}
      </div>
    )
  }
}
```
