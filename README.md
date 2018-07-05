# tf-guidelines

> Programing wisdom quotes were took from [twitter.com/codewisdom](https://twitter.com/codewisdom).

The purpose of this repository is to have explicit some guidelines when dealing with starting projects and how to code with a mindset that will help us in the future. Here are some rules that linters like ESLint or PEP8 can not recognize but as humans we need to he aware that coding is designing a system and a good design prevents errors.

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
    - [Pass the props](#pass-the-props)
    - [Composable elements](#composable-elements)
    - [Functional component vs class component](#functional-component-vs-class-component)
    - [Declarative programing in React](#declarative-programing-in-react)

---

## Starting new projects

> "Take care of your tools and they will take care of you."

When creating new projects the following checklist should be completed before start coding the actual requirements and sharing the codebase with the team:

- [ ] Follow the naming conventions.
  - GOOD: Platform repositories uses `tf-[name]` lowercase, so the new repository will be named `tf-cms`.
  - BAD: Break the convention with `TFCMS`, `tfcms`, or similar.
- [ ] Create `.gitignore` for all the operating-systems (MacOS|Windows|Linux).
  - GOOD: Use https://www.gitignore.io/api/linux,macos,windows
- [ ] Add to `.gitignore` the platform specific ignore blobs (Node.js, Python, Go, etc).
  - GOOD: Use https://www.gitignore.io
- [ ] **Setup the testing suite with at least one passing test** so serves as reference for the future development.
- [ ] Setup a code linter & formatter. This will help with the consistency of the code during it's development.
  - GOOD: Use [tokenfoundry/eslint-config](https://github.com/tokenfoundry/eslint-config).
- [ ] If you need to expose a PORT do not collide with the port of the other projects.
- [ ] Set the metadata information (`package.json`) versioning to `0.0.0`.
- [ ] Create `README.md` with at least this information:
  - [ ] Propuse of the repository and the relationship with the other repositories.
  - [ ] Requirements like: Node.js >=8, Yarn, Docker 1.8, etc.
  - [ ] `bash`/`shell` command line code to clone the repository.
  - [ ] The markdown output may be pretty and formated, but please use indentation lines, breaklines and write a beautiful markdown source code.
  - [ ] Prefer having a `.env` file (please add it to `.gitignore`).
    - GOOD: Use `dotenv` packages for Node.js, find similar for other languages.
  - [ ] For each environment (_development_, _production_, _testing_, _ci_):
    - [ ] Write the necessary environment variables. If the variable can be shared please inline in directly in the `README.md`, but private variables just leave it empty and write where/how to ask for that value.
    - [ ] Write the necessary commands to install and run the project.
    - [ ] Write where to access the running application or where is the output.
    - [ ] Write the common errors that may appear and how to fix it.

  ````md
  # tf-cms

  Manage content for the [tf-frontend](https://github.com/tokenfoundry/tf-frontend) that can't be stored ia SQL database.

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

  Run:
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

If the project plain text and doesn't involve team collaborations (like this), keep everything in `master` and create PRs as needed.

If the project is large and involves many people, the branching should be like:

- `master` branch: is the current production build
- `develop` branch: is the current staging build, this version should be stable enough.
- `feature/[name]` branches: has work in progress (WIP) code that will eventually get **squashed and merged** into `develop`.
- `feature/[name]-[subname]` branches has a subset of the work required for the main feature branch. This help the squad to collaborate and be able to track and review the team current progress and to identify problems beforehand.
- `chore/[name]` branches has work related to the health and refactoring of the code.
- `hotfix/[name]` branches has a correction of a detected bug.

It's important to mention that it's good to have an opened PR even before the branch is done because allows the team to review the code and detect problems _a priori_.

---

## Programing recommendations

> "The purpose of software engineering is to control complexity, not to create it."

### Make use of the built-in features

> "Any program is only as good as it is useful." - Linus Torvalds

It's common to try to write code fast and have features done with the current knowledge without questioning if what we know is correct or wrong.

If you feel _"Why i'm writing something long that is very common to any software or language?"_, probably someone already have the solution.

Always research and you will end up writing fewer and better lines of code:

```js
const registrations = [];

// BAD, six lines of inefficient code
const isApproved = true;
for (var i = 0; i < registrations.length; i++) {
  if (registrations["approved"] === false) { // please use `!registrations["approved"]`
    isApproved = false;
  }
}

// BAD, works but not correctly semantic and hard to understand
const isApproved = registrations.findIndex(r => !r["approved"]) > -1;

// GOOD, awesome! clear and semantic, you don't need extra comments.
const approved = registrations.every(r => r["approved"]);
```

### Copypaste versus abstraction

> "No abstraction is better than wrong abstraction."

Sometimes we feel necessity of writting a _class_ or any abstraction when dealing with some repetitive task. This may lead to complexity problems when the abstraction is made incorrectly, so sometime it's better to copy-paste code instead of writing the wrong abstraction.

Also it's good for the developer to avoid traveling from function to function. Try to keep everything close where it is needed.

```js
const keyBy = require("lodash/keyBy");

// BAD: we don't need abstraction for everything.
const getCreateBadIndexUsersBy = (users, id, name, insensitive) => {
  let result = null;
  if (insensitive) {
    users = users.map(u => u.toLowerCase());
  }
  // This function is a mess, i know. It is just an example.
  if (id || name) {
    result = id ? keyBy(users, id) : keyBy(users, name)
  }
  // ...
  return result;
}
const usersById = getCreateBadIndexUsersBy(users, "", null, false);
const usersByName = getCreateBadIndexUsersBy(users, null, "name", false);

// GOOD: it's clear and specific operations are handled by the developer.
const usersById = keyBy(users, "id");
const usersByName = keyBy(users.map(u => ... })), "name");
```

### Object properties

> "Explicit is better than implicit. Simple is better than complex, ..." -The Zen of Python

In Javascript you have three ways of accessing object properties:

1. `const { user } = this.props;`
    - Use this when you need a 1-5 variables of the object and try to avoid renaming.
2. `const user = this.props.user;`
3. `const user = this.props["user"];`
    - Use this when dealing with _external data_ as it were a JSON object (indeed is a JSON object so treat it like a JSON object, not a Javascript object).

**Using `["brackets"]` for external data helps the developer understand that we are dealing with data we don't have control on**. Also it's clear that it's a JSON object and not a internal Javascript property or value.

```js
// BAD, very long function, so many variables and are renamed.
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
  } = user; // is `user` an Javascript internal object or external data?
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
    age: Number(user["age"]), // never trust external data.
    ... // many other stuff
  }
}

// GOOD
class App extends Component {
  static defaultProps = {
    user: PropTypes.object.isRequired,
  };
  render() {
    // We know `user` is known because it's a prop.
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
    if (user["data"]["key"] != "0x") { // BAD, use `!==` not `!=`
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

// GOOD, plain control flow and add _firewalls_ so the "happy-path" return value is at the end.
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

If you are feeling that you are writing some much code, it's probably because you are doing something wrong. Here some tips:

- Use less temporal or aux variables.
- Build mental maps or representations easy for humans.
- Read the language documentation.
- Prefer functional programing style rather than imperative.
- Ask for help or recommendations.

And finally, don't copy-paste code if you don't understand it first.

```js
// GOOD, easy to read and the best performance.
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

When a prop changes so many styles, group them with `css` instead of using complex ternary operations.

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

### Pass the props

> "Make it correct, make it clear, make it concise, make it fast. In that order." – Wes Dyer

When creating a component, **always** pass the `...props`. Otherwise this can be frustrating to other dev trying to extend the component and realizing that nothing happens and he/she must spend time debugging your mistake.

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
 * CAN WE IMPROVE THIS? YES! Read "Composable elements" section
 */
```

> Read the [Composable elements](#composable-elements) next.

### Composable elements

> Read the [Pass the props](#pass-the-props) section first.

Want to create a sharable and re-usable component? Do not create an actual `React.Component` not `React.PureComponent` neither a `const Component = () => < />` component. **Create only styled-components**.

Because you will end over-propping your component.

**Just expose it's parts and let the developer compose and override the component part styles.**

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
const AvatarSquare = Avatar.extend`
  ...
`;
const AvatarImageSquare = Avatar.Image.extend`
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

Our style-guide enforces Functional Component over class-based components. But you are free to use class-based components when needed.

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

// GOOD-ish, but please read the `Declarative programing in React` section.
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

> Please read the [Declarative programing in React](#declarative-programing-in-react) next.

### Declarative programing in React

Avoid imperative programing in React. Props and methods should describe behavior and sould not force actions.

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
// GOOD, write prop types.
Button.propTypes = {
  disabled: PropTypes.bool,
};
// GOOD, add sane and explicit defaults.
Button.defaultProps = {
  disabled: false,
};

// ------------------------------------------------------------------

// BAD, this is imperative
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
      <div {...this.props}> // this good boi do not forget about props.
        {this.renderView(this.state.loading)}
      </div>
    )
  }
}

// PLEASE NO, don't create variables for components, makes the render method harder to read.
class App extends Component {
  state = {
    loading: false,
  };
  render() {
    const hardToRead = this.state.loading ?  <Loading /> : <MyComponent />;

    // ...

    return (
      <div {...this.props}> // this good boi do not forget about props.
        {hardToRead}
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
      <div {...this.props}> // this good boi do not forget about props.
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
