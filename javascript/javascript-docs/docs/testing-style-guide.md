---
id: testing-style-guide
title: Nielsen Testing style guide
sidebar_label: Testing style guide
---
![Nielsen Testing Style Guide](../static/img/logo.png)
This is the Nielsen Marketing Cloud Engineering team's style guide on web front-end testing.
> **Note**: The examples in this guide are using the [React](https://reactjs.org/) user interface library, in combination with [Jest](https://jestjs.io/) testing framework. We will share our opinions on different libraries for testing `React` components (Like [react-testing-library](https://github.com/testing-library/react-testing-library) which we also use for examples). For End-to-end testing we use [Cypress](https://www.cypress.io/). However, we believe that our suggested approach could be well used with other frameworks/libraries with some slight adjustments

## Basic Rules

#### AAA Pattern - Arrange Act Assert
We should structure our tests by the [AAA pattern](http://wiki.c2.com/?ArrangeActAssert), containing a visual separation between each block.
This will allow the reader to spend no time to figure out how our test works.
- Arrange: All the setup needed to happen to bring the system to the scenario the test aims to simulate. Adding DB records, mocking functions, creating objects and any other preperation your test needs.
- Act: Execute the unit under the test. This step usually contains one line of code.
- Assert: Ensure the received value satisfies the expectation.
This step usually contains one test **concept**. It may include more than one assertion.

Example component:
```javascript
function SearchBox({ value, onChange, disabled }) {
   return (
        <input
            placeholder='Search'
            onChange={onChange}
            value={value}
            disabled={diabled}/>
    );
}
```

✅ Do - Follow the AAA pattern and have visible separations between each block:
```javascript
test('SearchBox: Should not call onChange when input is disabled', () => {
  const onChange = jest.fn()
  const {getByPlaceholder} = render(<SearchBox
                                        value=''
                                        onChange={onChange}
                                        disabled/>)
  const domInput = getByPlaceholder('Search')

  fireEvent.change(domInput, { target: { value: '23' } })

  expect(onChange).toHaveBeenCalledTimes(0)
  expect(domInput.value).toBe('')
})
```

❌ Don't - Write in one block. It's harder to interpret and understand without diving into the code:
```javascript
test('SearchBox: Should not call onChange when input is disabled', () => {
  const onChange = jest.fn()
  const {getByPlaceholder} = render(<SearchBox
                                        value=''
                                        onChange={onChange}
                                        disabled/>)
  const domInput = getByPlaceholder('Search')
  fireEvent.change(domInput, { target: { value: '23' } })
  expect(onChange).toHaveBeenCalledTimes(0)
  expect(domInput.value).toBe('')
})
```

##### Act in React
> When writing UI tests, tasks like rendering, user events, or data fetching can be considered as “units” of interaction with a user interface. `React` provides a helper called act() that makes sure all updates related to these “units” have been processed and applied to the DOM before you make any assertions

Simply put, if you render or interact with an element, you need to wrap the code that may cause side-effects inside an act function.
The goal is to make your test run closer to what real users would experience.
If you feel like wrapping every interaction you make with `act` is an overhead, you can use [React-Testing-Library](https://testing-library.com/docs/react-testing-library/intro) which already wraps interactions with `act` for you.

Example Component:
```javascript
 function App() {
   let [counter, setCounter] = useState(0);
   useEffect(() => {
     setCounter(1);
   }, []);
   return counter;
 }
```
✅ Do - Wrap your interactions with `act` to guarantee that any state updates and enqueued effects will be executed:
```javascript
test("should render 1", () => {
  const el = document.createElement("div");
  act(() => {
    ReactDOM.render(<App />, el);
  });
  expect(el.innerHTML).toBe("1"); // this passes!
});
```
❌ Don't - Interact without wrapping your interaction with `act`:
```javascript
test("should render 1", () => {
  const el = document.createElement("div");
  ReactDOM.render(<App />, el);
  expect(el.innerHTML).toBe("1"); // this fails!
});
```
##### Async side effects in react
If your component triggers an XHR (in `useEffect` for example), you will want to wait for an answer (even when mocking APIs).
Wrapping your `render` with `act` won't do it in this case. In order to test your component with data, we suggest to mock the request and wait for the data to be shown on the screen.

Example component:
```javascript
 function UsersList({getUsers, users = [], isFetching}) {
   useEffect(() => {
     getUsers();
   }, []);
   return isFetching ?
        (
            <div>Loading...</div>
        ) :
        (
            <>
                <div>Users:</div>
                {
                    users.map(currUser => (
                        <div
                            key={currUser.id}
                            role='listitem'>
                                {currUser.name}
                        </div>
                    )
                }
            </>
        );
 }
```
✅ Do - Wait for an element to appear when waiting for an XHR request to finish:
```javascript
test("should render 3 users", () => {
    // React-Testing-Library wraps act with render
    render(<App />);

    await waitForElement(() => getByText('Users:'));
    expect(getByRole('listitem')).toHaveLength(3); // this passes
});
```
❌ Don't - Query the DOM without waiting for an indication that the async operation has finished:
```javascript
test("should render 1", () => {
    // React-Testing-Library wraps act with render
    render(<App />);
    expect(getByRole('listitem')).toHaveLength(3); // this fails
});
```

## Unit Testing

### React Component
We strongly encourage testing components' behavior rather than testing their implementation details. What does it mean? Generally, testing the behavior means making assertions about the outcome of a desired action, rather than asserting about the way the component achieved this outcome internally.

This means we can rewrite/refactor a component implementation and have its tests remain the same and not break.
If the test is tied to the way the component is implemented (e.g. by calling an internal function directly), changes in the implementation (e.g. renaming that function) would break the test even though the component still behaves the same.

>**Note**: Using `React Testing Library` helps us avoid writing implementation details tests, as opposed to `Enzyme` where it's much easier to test implementation details unintentionally.

#### Hooks Component

React hooks components are great to show why behavior testing is the way to go. The reason for it is that until lately we were used to only write class components, which are easy to test using implementation details tests.
This testing pattern will need change drastically in order to fit new functional components using hooks.

The following example demonstrates exactly how implementation details tests break easily, while behavior tests continue working even after the switch to React hooks.

Here is a simple class component, rendering a button and a click counter:
```javascript
export class ClickCounterClass extends React.Component {
    state = {
      count: 0
    }
    render() {
      return (
        <div>
          <p data-testid='counter-value'>{this.state.count}</p>
          <button data-testid='counter-button' onClick={() => this.setState(({count}) => ({count: count + 1}))}>
            Click me
          </button>
        </div>
      );
    }
  }
```
And here is a functional component that does the same thing using the `useState` hook:
```javascript
export const ClickCounterHooks = () => {
  const [count, setCount] = useState(0)
  return (
    <div>
      <p data-testid='counter-value'>{count}</p>
      <button data-testid='counter-button' onClick={() => setCount(count => count + 1)}>
          Click Me
      </button>
    </div>
  )
}
```
And now, lets test the outcome of clicking the button:

✅ Do - Test the behavior. Make an assertion about the actual DOM element that we expect to change as a result of the click:
```javascript
test('shows the correct amount of clicks', () => {
  const testMessage = 'Test Message'
  const { queryByText, getByTestId } = render(<ClickCounterClass />)
  
  fireEvent.click(getByTestId('counter-button'))
  
  expect(getByTestId('counter-value').textContent).toBe('1')
})
```
The above test will pass both if we render `<ClickCounterClass />` or if we render `<ClickCounterHooks />`

❌ Don't - Make assertions about details in the component's implemantation. They will eventually change and will wrongly cause the test to fail:
```javascript
test('Shows the correct amount of clicks', () => {
  const wrapper = shallow(<ClickCounterClass />)

  wrapper.find('button').simulate('click')

  expect(wrapper.instance().state.count).to.equal(1)
})
```
This above "Don't" example uses Enzyme's `shallow()` and `wrapper.instance()`. It asserts about the 'count' prop in the state which is an implementation detail.
This test passes when we render `<ClickCounterClass />`, but it fails when we change it to render `<ClickCounterHooks />`

#### Class Component

When testing the async functionality of a `Class Component`, we want to ensure that our side effect finished running before asserting.
This is important because we should only assert after we are sure all changes to the DOM were made.
We should also focus on testing the component functionality instead of testing it's implementation details. That way our tests are maintainable and insure, with high confidence, that are components behave as they should.

Example Login component:
```javascript
class Login extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      resolved: false,
      error: null
    };
  }

  handleSubmit = async(event) => {
    event.preventDefault();
    const { username, password } = event.target.elements;
    this.setState({ resolved: false, error: null });
    try{
        await login(username, password);
        this.setState({ resolved: true, error: null });
    } catch (error) {
        this.setState({ resolved: false, error: error.message});
    }
  };

  render() {
    return (
      <div>
        <form onSubmit={this.handleSubmit}>
          <div>
            <label htmlFor="username">Username</label>
            <input id="username" />
          </div>
          <div>
            <label htmlFor="password">Password</label>
            <input id="password" type="password" />
          </div>
          <button type="submit">Login</button>
        </form>
        {this.state.error ? (
          <div role="alert">{this.state.error.message}</div>
        ) : null}
        {this.state.resolved && <div role="alert">Congrats! You're in!</div>}
      </div>
    );
  }
}
```
✅ Do - Make sure to wait for the side effect to finish before selecting elements on the DOM and making assertions:
```javascript
test('allows the user to login successfully', async () => {
  jest.spyOn(window, 'login').mockImplementationOnce(() => {
    return Promise.resolve({})
  })
  const {getByLabelText, getByText, findByRole} = render(<Login />)

  fireEvent.change(getByLabelText(/username/i), {target: {value: 'chuck'}})
  fireEvent.change(getByLabelText(/password/i), {target: {value: 'norris'}})
  fireEvent.click(getByText(/submit/i))

  // wait for the side effect to finish before selecting elements on the dom
  const alert = await findByRole('alert')
  expect(alert).toHaveTextContent(/congrats/i)
})
```
❌ Don't - Select a DOM element without waiting for the side effect to finish. You might miss changes to the DOM:
```javascript
test('allows the user to login successfully', async () => {
  jest.spyOn(window, 'login').mockImplementationOnce(() => {
    return Promise.resolve({})
  })
  const {getByLabelText, getByText, findByRole} = render(<Login />)

  fireEvent.change(getByLabelText(/username/i), {target: {value: 'chuck'}})
  fireEvent.change(getByLabelText(/password/i), {target: {value: 'norris'}})
  fireEvent.click(getByText(/submit/i))

  // Selecting a dom element without waiting for the side effect to finish is wrong!
  const alert = findByRole('alert')
  expect(alert).toHaveTextContent(/congrats/i)
})
```
❌ Don't - Test state changes, as it is an implementation detail. That way refactoring your component might break your tests:
```javascript
test('allows the user to login successfully', async () => {
  jest.spyOn(window, 'login').mockImplementationOnce(() => {
    return Promise.resolve({})
  })

  const wrapper = mount(<Login/>)

  wrapper.find('#username').simulate('change', {target: {value: 'chuck'})
  wrapper.find('#password').simulate('change', {target: {value: 'norris'})
  wrapper.find('button[type=\"submit\"]').simulate('click')

  await new Promise(resolve => setImmediate(resolve))

  expect(wrapper.find('role=\"alert\"')).toContain('congrats')

  // Implementation detail, shouldn't be tested!
  expect(wrapper.state('resolved')).toBeTruthy()
})
```
❌ Don't - Use native element selectors, as it is also an implementation detail:
```javascript
test('login - implementation details', async () => {
  jest.spyOn(window, 'login').mockImplementationOnce(() => {
    return Promise.resolve({})
  })

  const wrapper = mount(<Login/>)

  // Implementation detail, shouldn't be tested!
  expect(wrapper.find('input[type=\"text\"]').length).toBe(1)
  expect(wrapper.find('input[type=\"password\"]').length).toBe(1)
})
```

#### Redux Connected Component

When testing the integration between a `Connected Component` and its `React Component`, we want to ensure that any change in the `React Component` interface (i.e. props) will cause the tests to fail.
The idea is to create a small compatability test between a `React Component` and `Connected Component` (which connects it with redux).

We want to avoid testing `mapStateToProps`/`mapDispatchToProps` isolated! In this case your test will pass even if the `React Component` props have changed.

Example redux connected component:
```javascript
const CounterComponent = ({ counter, increment, decrement }) => (
  <div>
    <button onClick={increment}>incrementBtn</button>
    <button onClick={decrement}>decrementBtn</button>
    <div data-testid="counter-element">{counter}</div>
  </div>
)

const mapStateToProps = state => ({
  counter: getCounterSelector(state)
})

const mapDispatchToProps = dispatch =>
  bindActionCreators({ increment, decrement }, dispatch)

const ConnectedComponent = connect(mapStateToProps, mapDispatchToProps)(CounterComponent)
```
✅ Do - Mock the action creators, and assert the expected actions to have been called:
```javascript
import { increment, decrement } from '..'
jest.mock('./MockActionCreators') // mock increment and decrement action creators

describe('redux connected component unit-test', () => {
  test('should integrate safe', () => {
    const { getByTestId, queryByText } = render(
      <Provider store={configureStore()}>
        <ConnectedComponent />
      </Provider>
    )

    fireEvent.click(queryByText('incrementBtn'))

    expect(increment).toHaveBeenCalledTimes(1)
  })
})
```
❌ Don't - Test the results of firing the redux action. This should be tested separately in dedicated reducer tests:
```javascript
describe('redux connected component unit-test', () => {
  test('should integrate safe', () => {
      const { getByTestId, queryByText } = render(
        <Provider store={configureStore()}>
          <ConnectedComponent />
        </Provider>
      )

      fireEvent.click(queryByText('incrementBtn'))

      expect(getByTestId('counter-element').textContent).toBe('1')
  })
})
```

### Custom Hooks

Sometimes, we need to create custom hooks that encapsulate relevant logic for multiple developers on our team.

If these hooks can be easily tested by testing a component that uses them, we prefer doing that, in order to keep with the behavioral testing approach.
If that's not the case, we encounter a problem where a hook can only be called inside the body of a functional component.
To test these cases, we use [react-hooks-testing-library](https://react-hooks-testing-library.com/)

Lets take a `useToggle` hook example:
```javascript
const useToggle = (initialValue) => {
  const [isOn, setIsOn] = useState(initialValue)
  const open = () => {
    setIsOn(true)
  }
  const close = () => {
    setIsOn(false)
  }
  const toggle = () => {
    setIsOn(prevIsOn => !prevIsOn)
  }
  return [isOn, toggle, open, close]
}
```
Testing it using `react-hooks-testing-library` should look like this:
```javascript
test('should toggle state on off', () => {
  const { result } = renderHook(() => useToggle(false))

  act(() => {
    const [, toggle] = result.current
    toggle()
  })

  const [isOpen] = result.current
  expect(isOpen).toBe(true)
})
```
Don't forget to wrap every interaction with `act`, so after the interaction finishes, you'll get the updated value!

### Proper use of `Date`

We want to mock the `Date` object itself, this way we know for sure that we will get the exact result for the given date any time. In addition, we ensure that our current timestamp will always return the same result instead of creating a new timestamp every test, which could lead to false negatives.

✅ Do - mock the `Date` object, not the libraries that use it:
``` javascript
const DATE_TO_USE = new Date('2018–01–30T12:34:56+00:00')
const _Date = new Date()

beforeAll(()=> {
	global.Date = jest.fn(()=> DATE_TO_USE)
	// Date methods
	global.Date.UTC = _Date.UTC
	global.Date.now = _Date.now
})

afterAll(()=> {
	global.Date = _Date
})
```
❌ Don't - mock external libraries that use the `Date` object:
``` javascript
// mocking moment.js for example
jest.mock('moment', ()=> ({
	// moment methods that you need
}))
```

## End-to-end Testing
> For End-to-end testing we use [Cypress](https://www.cypress.io/) Framework. However, our suggested concepts could be adopted with other End-to-End frameworks/test runners. 

The below diagram illustrates main blocks that involved in End-to-end test: [Test](#test), [Page Object](#page-object), [Flow](#flow)

![End-to-end Test diagram](../static/img/e2eDiagram.png)
   
##### Page Object

   - Responsible to encapsulate technical details (like css selectors, data attributes, etc`) to access and manipulate the elements of the tested application page
   - Provides an API for atomic interaction with the tested application page. This API is used by [Test](#test) 
   - Has no assertions
   - Can use another Page Object, it depends on the hierarchical structure of application pages 
   - Can optionally use *`Selectors`* module, that contains reusable css selectors definitions
   - Complex shared components should expose Page Objects for other consumers
   - You can refer [this article](https://martinfowler.com/bliki/PageObject.html) for more information about Page Object

Example of Page Object:    
``` javascript
class TestPageObject {
  
  getSearchInput () {
    return cy.get('[data-test-id="search-input"]')
  }

  typeSearchTerm (searchTerm) {
    this.getSearchInput().type(searchTerm)
  }
  
  getSearchResults () {
    return cy.get('[data-test-id="search-results-table"]')
  }
}
```
##### Flow
   - Common reusable logic that can be shared between tests (i.e login flow)
   - Uses one or more [Page Objects](#page-object) under the hood
   - Has no assertions
  
##### Test

  - Uses the [AAA pattern](#aaa-pattern---arrange-act-assert)
  - One and only one unit that __has assertions__ for testing components' integration and data integrity 
  - Uses one or more [Page Objects](#page-object) to access and interact with tested application page
  - Uses one or more [Flows](#flow) to implement ["Act"](#aaa-pattern---arrange-act-assert) part of the test
    
Here is a simple example of test that checks search data functionality and uses 'TestPageObject':    

``` javascript
import TestPageObject from './TestPageObject'
const testPageObject =  new TestPageObject()

it('should search data correctly', () => {
  
  testPageObject.typeSearchTerm('searchTerm')
  
  testPageObject.getSearchResults().should('contain', 'searchTerm')
})
``` 


## Find us

  - We have a Tech Blog! You can find it at [Medium](https://medium.com/nmc-techblog).

## Contributors

  - [Contributors](https://github.com/nielsen-oss/docs/graphs/contributors)

## Amendments

Feel free to submit pull requests and contribute to this style guide!

## License

(Apache License 2.0)

Copyright (c) 2020 Nielsen

**[⬆ back to top](#nielsen-testing-style-guide)**

