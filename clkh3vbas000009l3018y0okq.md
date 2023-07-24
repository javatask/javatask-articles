---
title: "AI-powered software development. Part 2-1.Tests Generation"
seoTitle: "AI Software Creation: Part 2-1 Test Gen"
seoDescription: "Optimize testing with ChatGPT and GitHub Copilot for unit, integration, performance tests, and updated library usage"
datePublished: Mon Jul 24 2023 16:51:57 GMT+0000 (Coordinated Universal Time)
cuid: clkh3vbas000009l3018y0okq
slug: ai-powered-software-development-part-2-1tests-generation
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1690217424961/14149f26-daf4-4d25-831a-f517fa9c4496.png
tags: ai, software-development, testing, qa, chatgpt

---

# Introduction
This part contains test generation uses cases for your existing code. This article is part of [AI-powered software development series](https://blog.javatask.dev/series/aipower) The majority of use cases are using ChatGPT. When GitHub Copilot is, used, it will explicitly be mentioned. A lot of tests are based on my demo repository https://github.com/javatask/graphql-serverless. This repository is part of [An efficient way to build your serverless microservices series](https://blog.javatask.dev/series/awssam)

> Note. At the time of writing the article, the capacity of ChatGPT was limited, so paid GitHubCopilot examples were added to have better experiences

# Structure of use cases
I will follow the next structure:
* Description
* ChatGPT Input
* ChatGPT Output
* Output (optional)
* Copilot Input
* Copilot Output

# Use cases

## Test generation

### Generate Unit Test with ChatGPT and Copilot

*Description*: Sample request to generate *ChatGPT Input:* "Generate Jest tests for the code"

```javascript
export const getState = (text) => {
  const state = `Hello ${text}`;
  console.log("State value:", state);
  // throw error on random invocation
  if (Math.random() > 0.5) {
    throw new Error("This is an random error");
  }
  return state;
};
```

*ChatGPT Output:*

```javascript
describe("getState", () => {
  test("returns the state string with text passed", () => {
    expect(getState("world")).toBe("Hello world");
  });

  test("throws an error on random invocation", () => {
    jest.spyOn(Math, "random").mockReturnValue(0.6);
    expect(getState).toThrow("This is an random error");
    Math.random.mockRestore();
  });
});
```

This test has an error in the first use case - `getState` function does not accept parameters, and you need to add spyOn Math.random to the first use case.  
Using `Math.random()` to throw error adds complexity that is not easy to handle by AI/ After human help, it works:

![]( align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673535784577/994f6153-7d08-40ee-b520-430fd3ec12b7.png align="center")

*Copilot Input* To get tips to copilot that you want to generate a test for the function, create file `state.test.js` in the `__tests__` directory of your NodeJS project. Then add comment `// generate tests for getState`, and you will get a result.

*Copilot Output*

```javascript
describe("getState", () => {
  // test 1
  it("should return 'Hello nothing' when no env variable is set", () => {
    expect(getState()).toBe("Hello nothing");
  });
  // test 2
  it("should return 'Hello test' when env variable is set", () => {
    process.env.TEST_VAR = "test";
    expect(getState()).toBe("Hello test");
  });
}
```
This test suite may fail because we have a random error in the business logic.   

### Generate Integration Test

*Description*: Sample request to generate *ChatGPT Input:* "Generate Jest tests for the code"

```javascript
export const getState = (text) => {
  const state = `Hello ${text}`;
  console.log("State value:", state);
  // throw error on random invocation
  if (Math.random() > 0.5) {
    throw new Error("This is an random error");
  }
  return state;
};
```

*ChatGPT Output:*

```javascript
describe("getState", () => {
  test("returns the state string with text passed", () => {
    expect(getState("world")).toBe("Hello world");
  });

  test("throws an error on random invocation", () => {
    jest.spyOn(Math, "random").mockReturnValue(0.6);
    expect(getState).toThrow("This is an random error");
    Math.random.mockRestore();
  });
});
```

This test has an error in the first use case - `getState` function does not accept parameters, and you need to add spyOn Math.random to the first use case.  
Using `Math.random()` to throw error adds complexity that is not easy to handle by AI/ After human help, it works:

![]( align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673535784577/994f6153-7d08-40ee-b520-430fd3ec12b7.png align="center")

*Copilot Input* To get tips to copilot that you want to generate a test for the function, create file `state.test.js` in the `__tests__` directory of your NodeJS project. Then add comment `// generate tests for getState`, and you will get a result.

*Copilot Output*

```javascript
describe("getState", () => {
  // test 1
  it("should return 'Hello nothing' when no env variable is set", () => {
    expect(getState()).toBe("Hello nothing");
  });
  // test 2
  it("should return 'Hello test' when env variable is set", () => {
    process.env.TEST_VAR = "test";
    expect(getState()).toBe("Hello test");
  });
}
```
This test suite may fail because we have a random error in the business logic.   

### Generate Performance Test
*Description:* To generate a test, it is worth explicitly mentioning the test framework to be used     
*ChatGPT Input:* Generate performance test script using k6 framework for this graphql schema type Query { state: String }    
*ChatGPT Output:*    
```javascript
import { makeExecutableSchema } from "graphql-tools";
import { typeDefs } from "../src/schema/state.js";
import { resolvers } from "../src/resolvers/state.js";
import gql from "graphql-tag";

const schema = makeExecutableSchema({ typeDefs, resolvers });

describe("GraphQL schema", () => {
  it("has a query for state", async () => {
    const query = gql`
      query {
        state
      }
    `;
    const result = await graphql(schema, query);
    expect(result).toEqual({
      data: {
        state: "Hello World"
      }
    });
  });
});
```   
Output:   Failed to show event after changing the syntax from CommonJS to EMS, tests fails due to absence of `graphql` tool. `graphql-tools` during 2022 migrated to new `@` [namespace](https://the-guild.dev/graphql/tools/docs/generate-schema). This shows that googling and reading throw the documentation is crucial even with AI tools.   
Working test due to [Apollo Blog](https://www.apollographql.com/docs/apollo-server/testing/testing/) is 
```javascript
import { strict as assert } from 'node:assert';
import { ApolloServer } from "@apollo/server";

import { typeDefs } from "../src/schema/state.js";
import { resolvers } from "../src/resolvers/state.js";

// This includes all tests for getAllItemsHandler()
describe("Graphql test", () => {
  it("should return Test nothing", async () => {
    const testServer = new ApolloServer({
      typeDefs,
      resolvers,
    });

    const response = await testServer.executeOperation({
      query: "query Test { state }"
    });

    // Note the use of Node's assert rather than Jest's expect; if using
    // TypeScript, `assert`` will appropriately narrow the type of `body`
    // and `expect` will not.

    assert(response.body.kind === "single");
    expect(response.body.singleResult.errors).toBeUndefined();
    expect(response.body.singleResult.data?.state).toBe("Hello nothing");
  });
});
```

>Note. You may ask why to force ChatGPT to use `graphql-tools` when it can use ApolloServer to generate integration tests. I asked explicitly about `graphql-tools` due to the new version released in 2022 which ChatGPT and Copilot do not know.

# Summary
ChatGPT and Copilot can help you write units and integration tests for existing tests. But be aware that they are limited to the knowledge that was trained on. As it was shown, both tools failed to generate tests using the latest `Apollo Server` or `graphql-tools` libs.
ChatGPT can help to write performance tests using different frameworks, for this article k6 framework was used as an example.
