---
id: api-reference-relay-ql
title: Relay.QL
layout: docs
category: API Reference
permalink: docs/api-reference-relay-ql.html
next: api-reference-relay-mutation
---

Relay fragments, mutations, and queries must be specified using ES6 template literals tagged with `Relay.QL`. For example:

```
var fragment = Relay.QL`
  fragment on User {
    name
  }
`;
```

To execute this code, Relay needs access to the schema - which can be too large to bundle inside the application. Instead, these `Relay.QL` template expressions are transpiled into JavaScript descriptions via the `babel-relay-plugin`. This schema information allows Relay to understand things like the types of field arguments, which fields are connections or lists, and how to efficiently refetch records from the server.

## Related APIs

`Relay.QL` objects are used by the following APIs:

<ul class="apiIndex">
  <li>
    <pre>() => Relay.QL`fragment on ...`</pre>
    Specify the data dependencies of a `Relay.Container` as GraphQL fragments.
  </li>
  <li>
    <pre>(Component) => Relay.QL`query ...`</pre>
    Specify the queries of a `Relay.Route`.
  </li>
  <li>
    <pre>Relay.QL`mutation { fieldName }`</pre>
    Specify the mutation field in a `Relay.Mutation`.
  </li>
  <li>
    <pre>var fragment = Relay.QL`fragment on ...`;</pre>
    Reusable fragments to compose within the above use cases.
  </li>
</ul>


## Fragment Composition

Fragments can be composed in one of two ways:

- Composing child component fragments in a parent fragment.
- Composing fragments defined as local variables.

### Container.getFragment()

Composing the fragments of child components is discussed in detail in the [Containers Guide](guides-containers.html), but here's a quick example:

```{5}
Relay.createContainer(Foo, {
  fragments: {
    bar: () => Relay.QL`
      fragment on Bar {
        ${ChildComponent.getFragment('childFragmentName')},
      }
    `,
  }
});
```

### Inline Fragments

Fragments may also compose other fragments that are assigned to local variables:

```{3-7,14,21}
// An inline fragment - useful in small quantities, but best not to share
// between modules.
var userFragment = Relay.QL`
  fragment on User {
    name,
  }
`;
Relay.createContainer(Story, {
  fragments: {
    bar: () => Relay.QL`
      fragment on Story {
        author {
          # Fetch the same information about the story's author ...
          ${userFragment},
        },
        comments {
          edges {
            node {
              author {
                # ... and the authors of the comments.
                ${userFragment},
              },
            },
          },
        },
      }
    `,
  }
});
```

Note that it is *highly* recommended that `Relay.Container`s define their own fragments and avoid sharing inline `var fragment = Relay.QL...` values between containers or files. If you find yourself wanting to share inline fragments, it's likely a sign that it's time to refactor and introduce a new container.
