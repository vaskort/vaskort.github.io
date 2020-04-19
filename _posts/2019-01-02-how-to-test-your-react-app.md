---
layout: post
title: How to test your React application
---

Whenever I start a new React project, I always find it helpful to look back at my previous projects to remind me how best to test **actions**, **components** and **reducers**. I’ll try to organise everything in one place — this blog post.

I will share the key points of the test process. All code examples are featured in this [dedicated repository](https://github.com/vaskort/react-testing), which you can clone and try out for yourself.

## The context of the app 

Let’s say you have a simple application that fetches a list of users from an API at the click of a button. These users are then stored to the Redux store. The JSX would be something like the below:

```jsx
// src/App.js
<div className="App">
  <button
    className="loadButton"
    onClick={this.props.getUsers}
    disabled={this.props.users.loading}
  >
    Load Users
  </button>
  {this.props.users.users.length > 0 && (
    <ul>
      {this.props.users.users.map(user => {
        return <User user={user} key={user.id} />;
      })}
    </ul>
  )}
  {this.props.users.error && (
    <div>
      A network error occurred
    </div>
  )}
</div>
```

The action creator that will get triggered would be:

```javascript
// src/actions/users.js
import axios from "axios";

export const getUsers = () => dispatch => {
  dispatch({
    type: "GET_USERS",
    payload: axios.get("https://jsonplaceholder.typicode.com/users")
  });
};
```

And the reducer that will handle this action would be:

```javascript
// src/reducers/users.js
export default (state = {}, action) => {
  switch (action.type) {
    case "GET_USERS_FULFILLED":
      return {
        users: action.payload.data
      };
    default:
      return state;
  }
};
```

Pretty straight forward right? But how would you test everything in this scenario?
Let's start by testing the action creator.

## Testing the Action Creator

```javascript
// src/actions/users.js
import axios from "axios";

export const getUsers = () => dispatch => {
  dispatch({
    type: "GET_USERS",
    payload: axios.get("https://jsonplaceholder.typicode.com/users")
  });
};
```
There are two important things about this code — that I like using the `axios` package and that I like using the `redux-promise-middleware package`.

Whenever you dispatch an action with a promise as its payload using `redux-promise-middleware`, it will immediately dispatch `GET_USERS_PENDING` and when your promise is finally settled it will dispatch either `GET_USERS_FULFILLED` or `GET_USERS_REJECTED`. That helps writing less code as you don’t have to explicitly write action creators for these states.

We’ll use Facebook’s Jest to write our tests because it gives you this ‘no-configuration’ experience, and it’s easier to mock libraries like `axios`.

What we need to do now is to mock `axios`. We don’t really want to fire an http request to our servers even if we are pointing to a development environment. It can confuse our analytics, will add unnecessary traffic and also make our tests run slower.

There’s a quick way to mock libraries in Jest, we just have to create an `axios.js` file under `src/__mocks__`:

```javascript
// axios.js
export default {
  get: jest.fn(() => Promise.resolve({ data: "mocked" }))
};
```

Then we can mock what `axios.get` function returns differently inside our tests depending on our test case. If you want to mock the `post` method of `axios`, just add a new method called `post` and return whatever you want.

Now we’re ready to create our test file to test the action creator, `src/actions/users.test.js`. The first part of the file would be something like:

```javascript
// src/actions/users.test.js 
import mockAxios from "axios";
import configureMockStore from "redux-mock-store";
import thunk from "redux-thunk";
import promiseMiddleware from "redux-promise-middleware";
import { getUsers } from "./users";

const mockStore = configureMockStore([thunk, promiseMiddleware()]);

describe("User Actions", () => {
  let store;

  beforeEach(() => {
    store = mockStore({
      users: {}
    });
  });
```

A key library here is the `redux-mock-store` library, which will help us mock the Redux store but also to test Redux async action creators and middleware. We’re initialising as normal, passing any middleware that we need. We need `thunk`(so we can return promises as our value in our payload property in our actions) and `promiseMiddleware` so the expected actions types are being dispatched (`_FULFILLED`, `_REJECTED` etc).

Inside the `beforeEach` function we reset the value of our store, so we don’t have any unexpected results in our assertions. If you’re not familiar with the helper functions that Jest provides, you might want to look here: [https://jestjs.io/docs/en/setup-teardown](https://jestjs.io/docs/en/setup-teardown).

Now, let's add our actual test:

```javascript
describe("getUsers action creator", () => {
  it("dispatches GET_USERS action and returns data on success", async () => {
    mockAxios.get.mockImplementationOnce(() =>
      Promise.resolve({
        data: [{ id: 1, name: "Vasilis" }]
      })
    );

    await store.dispatch(getUsers());
    const actions = store.getActions();
    // [ { type: "GET_USERS_PENDING" },
    //   { type: "GET_USERS_FULFILLED", payload: { data: [Array] } } 
    // ]

    expect.assertions(3);
    expect(actions[0].type).toEqual("GET_USERS_PENDING");
    expect(actions[1].type).toEqual("GET_USERS_FULFILLED");
    expect(actions[1].payload.data[0].name).toEqual("Vasilis");
  });
});
```
The first thing we’re doing here is mocking what axios.get function will return in this particular test. We’re using Jest’s [mockImplementationOnce](https://jestjs.io/docs/en/mock-functions#mock-implementations) to return a Promise.resolve along with some mock data. Then we’re dispatching our action creator (`getUsers`) and making any assertions we feel are necessary. We expect the action types but also the data that the second action type is returning. Notice we used [`getActions`](https://github.com/dmitry-zaets/redux-mock-store#asynchronous-actions), this is a function from `redux-mock-store` that gives you all the action types that have been dispatched.

Now that we have tested our success state, we also need to test what happens when things don’t go so well and we receive an error state. It would be quite similar and something like this:

```javascript
it("dispatches GET_USERS action and returns an error", async () => {
  mockAxios.get.mockImplementationOnce(() =>
    Promise.reject({
      error: "Something bad happened :("
    })
  );
  
  try { 
    await store.dispatch(getUsers());
  } catch {
    const actions = store.getActions();

    expect.assertions(3);
    expect(actions[0].type).toEqual("GET_USERS_PENDING");
    expect(actions[1].type).toEqual("GET_USERS_REJECTED");
    expect(actions[1].payload.error).toEqual("Something bad happened :(");
  }
});
```

The only difference is that we’re mocking our implementation differently and running the assertions inside the `catch` block.

Hopefully at this stage when we run `yarn test` or `npm test` we'll see something like this in the console:

![Action Screenshot]({{ site.url }}/images/action-test.png)

## Testing the Reducer

This is much more straightforward compared to testing actions, but arguably more important for our application because it’s where our state changes.

The main idea is to assert what the reducer returns when a specific action is being passed.

```javascript
// src/reducers/users.js
export default (state = {
  users: [],
  loading: false,
  error: false
}, action) => {
  switch (action.type) {
    case "GET_USERS_PENDING":
      return Object.assign({}, state, {
        loading: true,
      });
    case "GET_USERS_FULFILLED":
      return Object.assign({}, state, {
        users: action.payload.data,
        loading: false,
      });
    case "GET_USERS_REJECTED":
      return Object.assign({}, state, {
        loading: false,
        error: true
      });
    default:
      return state;
  }
};
```

Let's talk about the key points here: 

- We set a default state in case one is not passed when the reducer is called and inside the function we return the new state depending on which action happened.

- `GET_USERS_PENDING`: It sets `loading` to `true` so we can disable the button while we wait for the data to be returned. You can show a loading icon or whatever you want using this property. 

- `GET_USERS_FULFILLED`:  Our data was returned successfully so we store them in the `users` property and set `loading` back to `false`.

- `GET_USERS_REJECTED`: We set the `loading` to `false` and `error` to `true` so we can show the network error.

As this is the simplest solution, this will be our first test.

```javascript
// src/reducers/users.test.js
import users from './users';

describe('users Reducer', () => {
  const initialState = {
    users: [],
    loading: false,
    error: false
  };

  it('returns the initial state when an action type is not passed', () => {
    const reducer = users(undefined, {});

    expect(reducer).toEqual(initialState);
  });
});
```

We call the reducer with an undefined state and without an action type. We expect this to be equal with the initial state we have in our reducer.

The next test would be the one testing the `GET_USERS_PENDING` action.

```javascript
// src/reducers/users.test.js
it('handles GET_USERS_PENDING as expected', () => {
  const reducer = users(initialState, { type: "GET_USERS_PENDING" });

  expect(reducer).toEqual({
    users: [],
    loading: true,
    error: false
  });
});
```

We expect the `loading` property to have changed to `true` (and the `users` to still be empty).

The one for `GET_USERS_FULFILLED` action would be:

```javascript
it("handles GET_USERS_FULFILLED as expected", () => {
    const reducer = users(initialState, {
      type: "GET_USERS_FULFILLED",
      payload: {
        data: [
          {
            id: 1,
            name: "foo"
          }
        ]
      }
    });

    expect(reducer).toEqual({
      users: [
        {
          id: 1,
          name: "foo"
        }
      ],
      loading: false,
      error: false
    });
  });
```

We pass the payload in our action object and we expect this to be what the reducer returns in the users property. We also expect the `loading` property to have ‘switched’ to `false`.

I'll let you write the `GET_USERS_REJECTED` test yourself :grimacing:.

Now that we have tested our **reducer** and our **actions**, there’s only one thing that needs to be tested — our component.

## Testing the Component

Before starting this we need to add some configuration.

For testing components I prefer using Enzyme. It provides a clean API that is similar to jQuery.

So let's install `enzyme-adapter-react-16` and `enzyme`.

```bash
yarn add enzyme-adapter-react-16 enzyme --dev
```

At this point we need a setup file for our tests. In a project that is not a `create-react-app`,you can define the path of this file in your package.json like below:

```javascript
"jest": {
  "setupTestFrameworkScriptFile": "<rootDir>/test/setup/"
}
```

but in a `create-react-app` application you just need to create a file named `setupTests.js` in your `src` folder. Let's create it and add the following code:

```javascript
// src/setupTests.js
import { configure } from 'enzyme';
import Adapter from 'enzyme-adapter-react-16';

configure({ adapter: new Adapter() });
```

As well as installing Enzyme, you also need to install an adapter for the version of React you’re using. In this case, we’re using React version 16. For more information click here: https://airbnb.io/enzyme/docs/installation/index.html

We should now be ready to write our first component test. Let’s create a file under src with name App.test.js and add the following:

```javascript
// src/App.test.js
import React from 'react';
import { App } from './App';
import { shallow } from 'enzyme';
import renderer from 'react-test-renderer';

describe('App Component', () => {
  let props;

  beforeEach(() => {
    props = {
      users: {
        users: [],
        loading: false,
        error: false
      }
    };
  });

  it('renders without crashing', () => {
    const tree = renderer.create(<App {...props} />)

    expect(tree.toJSON()).toMatchSnapshot();
  });
});
```

When we run this test, it will create a new snapshot. Snapshots are an additional tool to have in your testing strategy as they will ‘notify’ you when your markup changes. There have been times when I haven’t updated my snapshots after changing the markup of a component. When my tests were failing, it was an extra confirmation step if I really wanted that markup change. Aside from being notified, when your markup changes you can get more value from snapshots. You can assert that an error copy is shown when a network error occurs.

Some important notes on the above test are:

- We are importing the named App component and not the default export.

```javascript
import { App } from './App';
```

We don’t really need to test the connected component as we can pass the props manually instead of creating a mock store and passing the whole object. I was a little sceptical about having two exports — the decorated and the undecorated one — but this is something that is also recommended in the Redux docs: [https://redux.js.org/recipes/writing-tests#connected-components](https://redux.js.org/recipes/writing-tests#connected-components)

- We’re using the React’s test renderer to create snapshots instead of Enzyme’s shallow. That’s a personal preference — as with the test renderer you don’t have to use a snapshot serialiser to make your snapshots readable, you can use the test renderer’s [`toJSON` function](https://reactjs.org/docs/test-renderer.html#testrenderertojson).

Now let’s create another snapshot test where we’ll test that the expected error copy was shown:

```javascript
it('renders an error message when a network error occurs', () => {
  props.users.error = true;
  const tree = renderer.create(<App {...props} />)

  expect(tree.toJSON()).toMatchSnapshot();
});
```

In our third test we’ll use Enzyme’s shallow. We’re going to mock the getUsers function and make sure it’s being called when we click the button.

```javascript
it("calls the getUsers function when the button is clicked", () => {
  props.getUsers = jest.fn();
  const wrapper = shallow(<App {...props} />);
  const spy = jest.spyOn(wrapper.instance().props, "getUsers");

  wrapper.find("button").simulate("click");
  expect(spy).toHaveBeenCalled();
});
```

The code should be self-explanatory but the key points are summarised below:

- Mocking `getUsers` with Jest's `jest.fn()`.
- Spying `getUsers` (note the correct syntax there).
- Using `find` and `simulate` methods from Enzyme (it's like jQuery right?).
- Asserting that spy has been called with Jest's `toHaveBeenCalled`.

Now let's test that the `User` component get's rendered when we have some users.

```javascript
it("renders the User correctly", () => {
  props.users.users = [
    {
      id: 1,
      name: "foo"
    }
  ]
  const wrapper = shallow(<App {...props} />);

  expect(wrapper.find("User").length).toBe(1);
});
```

You could test the `User` component the same way we did for the `App` component.

## Summary

Theoretically, you now know how to test most aspects of your React application. I believe it would be valuable to clone the [repository](https://github.com/vaskort/react-testing) and try to write some more tests, create new snapshots or even try to break it. Have fun!

### Helpful links

Project's GitHub url: [https://github.com/vaskort/react-testing](https://github.com/vaskort/react-testing)

- [https://www.leighhalliday.com/mocking-axios-in-jest-testing-async-functions](https://www.leighhalliday.com/mocking-axios-in-jest-testing-async-functions)
- [https://www.reddit.com/r/reactjs/comments/7eiyh8/is_enzymetojson_still_required_for_jest_snapshot/](https://www.reddit.com/r/reactjs/comments/7eiyh8/is_enzymetojson_still_required_for_jest_snapshot/)
- [https://reactjs.org/docs/test-renderer.html](https://reactjs.org/docs/test-renderer.html)
- [https://jestjs.io/docs/en/api](https://jestjs.io/docs/en/api)
- [https://github.com/airbnb/enzyme/issues/1647](https://github.com/airbnb/enzyme/issues/1647)