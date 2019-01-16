---
layout: post
title: How to test your React application
---

The reason I decided to write this blog post is because on every new project when I start writing unit tests I find it helpful to look back on older projects on how I've tested my **actions**, my **components** or my **reducers**. I'll try to organise everything in one place, this blog post.

I will share the key points of the test process. In the end you will find a [GitHub url](https://github.com/vaskort/react-testing) with the full code so you can clone and play.

## The context of the app 

Let's say you have a simple application where when you press a button it triggers an async action which loads a list of users from an API. These users are then stored to the redux store. The JSX would be something like the below:

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
      A network error occured
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

Let see again the action creator:

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

There are two important things on this code.
One is that I like using the `axios` package and two that I like using the `redux-promise-middleware` package.  

What `redux-promise-middleware` basically is doing is whenever you dispatch a promise as the value of the payload property of the action then it will immediately dispatch `GET_USERS_PENDING` and when your promise is finally settled it will dispatch either `GET_USERS_FULFILLED` or `GET_USERS_REJECTED`. I like using it because it helps me not to write boilerplate code.

We will use Facebook's Jest to write our tests because it gives you this "no-configuration" experience as Facebook claims but also because it's easier to mock libraries like `axios`.

What we need to do now is to mock `axios`. Why? Because we don't really want to fire an http request to our servers even if we are pointing to a development environment. It can mess our analytics, it will add unnecessary traffic but also it will make our tests run slower.

There is a quick way to mock libraries in Jest, we just have to create an `axios.js` file under `src/__mocks__`:

```javascript
// axios.js
export default {
  get: jest.fn(() => Promise.resolve({ data: "mocked" }))
};
```

We can then mock what axios.get functions returns differently inside our tests as every test would require. The same thing would be if you wanted to mock the `post` method of `axios`, just add a new method returning whatever you want.

Now we are ready to create our test file to test the action creator, `src/actions/users.test.js`. The first part of the file would be something like:

```javascript
import mockAxios from "axios";
import configureMockStore from 'redux-mock-store';
import thunk from 'redux-thunk';
import promiseMiddleware from 'redux-promise-middleware';
import { getUsers } from "./users";

const mockStore = configureMockStore([thunk, promiseMiddleware()]);

describe('User Actions', () => {
  let store;

  beforeEach(() => {
    store = mockStore({
      users: {}
    });
  });
```

A key library here is the `redux-mock-store` library which will help us mock the redux store but also to test Redux async action creators and middleware. We are initializing as normal passing any middleware that we need. We need `thunk` (so we can return promises as our value in our payload property in our actions) and `promiseMiddleware` so the expected actions types are being dispatched (`_FULFILLED, _REJECTED` etc).

Inside the `beforeEach` function we reset the value of our store so we don't have any unexpected results in our assertions. If you're not familiar with all these helper functions that Jest provides you might take a quick look here: [https://jestjs.io/docs/en/setup-teardown](https://jestjs.io/docs/en/setup-teardown).

Now, let's add our actual test:

```javascript
describe('getUsers action creator', () => {
  it('tests GET_USERS action and that returns data on success', async () => {
    mockAxios.get.mockImplementationOnce(() =>
      Promise.resolve({
        data: [{ id: 1, name: "Vasilis" }]
      })
    );

    await store.dispatch(getUsers());
    const actions = store.getActions();
    // [ { type: 'GET_USERS_PENDING' },
    //   { type: 'GET_USERS_FULFILLED', payload: { data: [Array] } } 
    // ]

    expect.assertions(3);
    expect(actions[0].type).toEqual("GET_USERS_PENDING");
    expect(actions[1].type).toEqual("GET_USERS_FULFILLED");
    expect(actions[1].payload.data[0].name).toEqual("Vasilis");
  });
});
```

So the first thing we are doing here is that we are mocking what axios.get function will return in this particular test. We are using Jest's [mockImplementationOnce](https://jestjs.io/docs/en/mock-functions#mock-implementations) to return a Promise.resolve along with some mock data. Then we are dispatching our action creator (`getUsers`) and inside the `then` function of the promise we make any assertions we feel are necessary. In our case we expect the action types but also the data that the second action type is returning. Notice we used [`getActions`](https://github.com/dmitry-zaets/redux-mock-store#asynchronous-actions), this is a function from `redux-mock-store` that gives you all the action types that got dispatched.

Now that we tested the successful case let's also test when things won't go well after trying to get users from our API. It would be quite similar and would be something like this:

```javascript
it("tests GET_USERS action and that returns an error", async () => {
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

The only difference is that we are mocking our implementation differently and notice using `catch` instead of `then` because our Promise gets rejected at this case.

Hopefully at this stage when we run `yarn test` or `npm test` we'll see something like this in the console:

![Action Screenshot]({{ site.url }}/images/action-test.png)

## Testing the Reducer

Testing our reducers is much more straightforward comparing to testing actions but its maybe more important for our application because its where our state changes.

The main idea is to assert what the reducer returns when a specific action and is being passed.

Let's see our reducer before we start writing a test for it:

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

- `GET_USERS_PENDING`: It sets `loading` to `true` so we can disable the button while we wait the data to be returned. You can show a loading icon or whatever you want using this property.  

- `GET_USERS_FULFILLED`: Our data got returned successfully so we store them in the `users` property and set `loading` back to `false`.  

- `GET_USERS_REJECTED`: Nothing fancy here we just set the `loading` to `false` and `error` to `true` so we can show the network error.

If the action type was not any of the above just return the default state. And that's basically would be our first test.

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
No surprises here, we expect the `loading` property to have changed to true (and the `users` to still be empty).

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

We pass the payload in our action object and we expect this to be what the reducer returns in the users property. We also expect the `loading` property to have "switched" to `false`.

I'll let you write the `GET_USERS_REJECTED` test yourself :grimacing:.

Now that we tested our **reducer** and our **actions** there's only one thing needs to be tested, our component.

## Testing the Component

Before starting with the component testing we need to add some configuration.

Generally for testing components I prefer using Enzyme. It provides a clean API that is similar to jQuery.

So let's install `enzyme-adapter-react-16` and `enzyme`.

```bash
yarn add enzyme-adapter-react-16 enzyme --dev
```

At this point we need a setup file for our tests. In a project that is not a `create-react-app` you can define the path of this file in your package.json like below: 

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

That is because Enzyme requires an adapter to be configured. Take a look at the docs if you want more info about this: https://airbnb.io/enzyme/docs/installation/index.html

At this point we should be ready to write our first component test. Let's create a file under src with name App.test.js and add the following:

```javascript
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

When we run this test it will create a new snapshot. Snapshots are an additional tool to have in your testing strategy as they will "notify" you when your markup changes. In the real world there were many times where I didn't update my snapshots after I changed the markup of a component, then when my tests were failing it was like an extra confirmation step if I really wanted that markup change. Apart from getting notified when your markup changes you can get more value from snapshots. You can assert that an error copy is shown when a network error occurs.

Some important notes on the above test are:

- We are importing the named App component and not the default export.  
```javascript
import { App } from './App';
```
We don't really need to test the connected component as we can pass the props manually instead of creating a mock store and passing the whole object. I was a bit sceptical in the beginning having two exports, the decorated and the undecorated one but this is something that is also recommended in the Redux docs: [https://redux.js.org/recipes/writing-tests#connected-components](https://redux.js.org/recipes/writing-tests#connected-components)

- We are using the React's test renderer to create snapshots instead of enzyme's shallow. That's a personal preference as with the test renderer you don't have to use a snapshot serialiser to make your snapshots readable, you can just use the test renderer's [`toJSON` function](https://reactjs.org/docs/test-renderer.html#testrenderertojson).

Now let's create another snapshot test where we will test that the expected error copy was shown:

```javascript
it('renders an error message when a network error occurs', () => {
  props.users.error = true;
  const tree = renderer.create(<App {...props} />)

  expect(tree.toJSON()).toMatchSnapshot();
});
```

In our third test we'll use Enzyme's shallow. We are going to mock the getUsers function and make sure its being called when we click the button.

```javascript
it('calls the getUsers function when the button is clicked', () => {
  props.getUsers = jest.fn();
  const wrapper = shallow(<App {...props} />);
  const spy = jest.spyOn(wrapper.instance().props, 'getUsers');

  wrapper.find('button').simulate('click');
  expect(spy).toHaveBeenCalled();
});
```

The code should be self explanatory but the key points are:

- Mocking `getUsers` with Jest's `jest.fn()`.
- Spying `getUsers` (note the correct syntax there).
- Using `find` and `simulate` methods from Enzyme (it's like jQuery right?).
- Asserting that spy has been called with Jest's `toHaveBeenCalled`.

Now let's test that the `User` component get's rendered when we have some users.

```javascript
it('renders the User correctly', () => {
  props.users.users = [
    {
      id: 1,
      name: 'foo'
    }
  ]
  const wrapper = shallow(<App {...props} />);

  expect(wrapper.find('User').length).toBe(1);
});
```

With the same strategy you could test the `User` component the same way we did for the `App` component.


### Helpful links

Project's GitHub url: [https://github.com/vaskort/react-testing](https://github.com/vaskort/react-testing)

- [https://www.leighhalliday.com/mocking-axios-in-jest-testing-async-functions](https://www.leighhalliday.com/mocking-axios-in-jest-testing-async-functions)
- [https://www.reddit.com/r/reactjs/comments/7eiyh8/is_enzymetojson_still_required_for_jest_snapshot/](https://www.reddit.com/r/reactjs/comments/7eiyh8/is_enzymetojson_still_required_for_jest_snapshot/)
- [https://reactjs.org/docs/test-renderer.html](https://reactjs.org/docs/test-renderer.html)
- [https://jestjs.io/docs/en/api](https://jestjs.io/docs/en/api)
- [https://github.com/airbnb/enzyme/issues/1647](https://github.com/airbnb/enzyme/issues/1647)