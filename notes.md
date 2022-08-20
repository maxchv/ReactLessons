# Reat notes

File `components/newTag.js`:

```js
import React from "react";

function newTag(props) {
  return (
    <>
      <h3 className="name">{props.children}</h3>
    </>
  );
}

export default newTag;
```

## Higher order components

```js
const makeGreen = (BaseComponent) => (props) => {
  const addGreen = {
    style: {
      color: "green",
    },
  };
  const newProps = {
    ...props,
    ...addGreen,
  };
  return <BaseComponent {...newProps} />;
};

const GreenNameTag = makeGreen(NameTag);
```

## Use State Hook

```js
import React, { useState } from "react";

function Component() {
  const [age, setAge] = useState(21);

  const ageUpHandle = () => {
    setAge(age + 1);
  };

  return (
    <div className="App">
      <header className="App-header">
        <h1>Use State Hook</h1>
        <h2>Age: {age}</h2>
        <button onClick={ageUpHandle}>Age up</button>
      </header>
    </div>
  );
}
```

## How does the hook work?

```js
function useState(init) {
  let _val = init;

  return [
    _val,
    (newVal) => {
      _val = newVal;
    },
  ];
}
```

But it doesn't work! Whe should do next things.

```js
const React = (function () {
  let _val;
  return {
    render(Component) {
      const Comp = Component();
      Comp.render();
      return Comp;
    },
    useState(init) {
      _val = _val || init;

      function setState(newVal) {
        _val = newVal;
      }

      return [_val, setState];
    },
  };
})();

function AgeComp() {
  const [age, setAge] = React.useState(21);
  return {
    render() {
      console.log(age);
    },
    ageUp() {
      setAge(age + 1);
    },
  };
}

let App = React.render(AgeComp);
App.ageUp();
React.render(AgeComp);
```

## Hooks rules

- Don't call inside loop
- Don't call inside condition
- Don't call inside nested function
- Alway use hooks at the top of the functon
- Only call hooks from react function

## Complex state and manage lists

```js
const initialName = [
  { firstName: "John", lastName: "Johnson" },
  { firstName: "Peter", lastName: "Peterson" },
  { firstName: "Jill", lastName: "Jillson" },
];

function Component() {
  const [names, setNames] = useState(initialNames);

  render <>
    <div className="App">
        <header className="App-header">
            {
                names.map((name, i) => {
                    return <NameTag firstName={name.firstName} lastName={name.lastName} key={`${i}${name.firstName}${name.lastName}`} />
                })
            }
        </header>
    </div>
  </>;
}
```

For key it is usefull use `Symbol()`

## Adding events

```js

function App() {
    const [names, setNames] = useState(initialNames);

    const removeHandler = (e) => {
        const copyList =  names.filter((name) => name.firstName == 'John'); //[...names];

        setList(copyList);
    };

    return (
        ...
        <button onClick={removeHandler}></button>
    );
}

```

## Component Communication

File `components/nameTag.js`:

```js
function NameTag(props) {
  return (
    <>
      <h2>First Name: {props.firstName}</h2>
      <h2>Last Name: {props.lastName}</h2>
      <button onClick={(e) => props.onClick(e, props)}>Delete</button>
    </>
  );
}
```

```js

function App() {
    ...
    const removeName = (e, name) {
        console.log(e.target);

        const newNames = names.filter(n => n.firstName == name.firstName && n.lastName == name.lastName);
        setNames(newNames);
    }

    return <>
        ...
        {
            names.map((n, i) => {
                return <NameTag firstName={n.firstName} lastName={n.lastName} onClick={removeName} />
            })
        }
        ...
    </>
}
```

## Custom Hooks

File: `src/hooks/useList.js`;

```js
import { useState } from "react";

function useList(init) {
  const [list, setList] = useState(init);

  return {
    list,
    removeItem(name) {
      const filtredList = list.filter((v) => v.name === name);
      setList(filtredList);
    },
    saveItem(index, newName) {
      const copyList = [...list];
      copyList[index].name = newName;
    },
  };
}

export default useList;
```

Use custom hooks:

```js
...
import useList from './hooks/useList';

const initList = [
  ...
];
...
function App() {
  const [editable, setEditable] = useState(false);
  const items = useList(initList);

  function removeItemHandle(e) {
    items.removeItem(e.target.name)
  }

  function keyPressHandle(e, i) {
    if(e.key === 'Enter') {
      setEditable(!editable);
      items.saveItem(i, e.target.value);
    }
  }

  function makeEditableHandle() {
    setEditable(true);
  }

  return (
    <div className="App">
      <div className="App-header">
        <h2>Grocery List</h2>
        {
          items.list.map((v, k) => {
            return <Item
                    key={`${k}${v.name}${v.calories}`}
                    item={v}
                    onClick={removeItemHandle}
                    editable={editable}
                    onDoubleClick={makeEditableHandle}
                    onKeyPress={keyPressHandle}
                    index={k}
                    />
          })
        }
      </div>
    </div>
  )
}
```

## onChange event

```js
import React, { useState } from "react";

function App() {
  const [name, setName] = useState("");
  const [income, setIncome] = useState("");

  function handleNameChange(e) {
    setName(e.target.value);
  }

  function handleIncomeChange(e) {
    setIncome(e.target.value);
  }

  function onSubmitHandle() {
    console.log("state = ", name, income);
  }

  return (
    <div className="App">
      <header className="App-header">
        <form onSubmit={onSubmitHandle}>
          <div>
            <label>
              Name:
              <input value={name} onChange={handleNameChange} />
            </label>
          </div>
          <div>
            <label>
              Income:
              <select value={income} onChange={handleIncomeChange}>
                <option name="high">Hight</option>
                <option name="mid">Mid</option>
                <option name="low">Low</option>
              </select>
            </label>
          </div>
          <input type="submit" value="submit" />
        </form>
      </header>
    </div>
  );
}
```

## use Refs hook

```js
import React, {userEffect, useRef} from 'react';

const formFieldStyel = {
  display: "flex",
  flex-direction: "row",
  marginBottom: "5px",
  justifyContent: 'space-between',
  width: '300px'
};

function App() {
  const nameRef = useRef();
  const ageRef = useRef();
  const marriedRef = useRef();
  const sumbitRef = useRef();

  useEffect(() => {
    nameRef.current.focus();
  }, [])

  function keyPressHandle(e) {
    if(e.key === 'Enter') {
      if(e.target.id == 'name') {
        ageRef.current.focus();
      }
      if(e.target.id == 'age') {
        marriedRef.current.focus();
      }
      if(e.targed.id == 'married') {
        submitRef.current.focus();
      }
    }
  }

  return (
    <div className="App">
      <header className="App-header">
        <h3>UseRefs Hook</h3>
        <div style={formFieldStyle}>
          <label htmlFor="name">Name</label>
          <input id="name" ref={nameRef} onKeyDown={keyPressHandle} />
        </div>
        <div style={formFieldStyle}>
          <label htmlFor="age">Age</label>
          <input id="age" type="number" ref={ageRef} onKeyDown={keyPressHandle} />
        </div>
        <div style={formFieldStyle}>
          <label htmlFor="married">Married?</label>
          <input id="married" type="checkbox" ref={marriedRef} onKeyDown={keyPressHandle} />
        </div>
        <button ref={submitRef}>Submit</button>
      </header>
    </div>
  );
}
```

## ForwardRef

File `src/components/Input.js`

```js
import React from "react";

function Input({ placeholder, style }, ref) {
  return <input placeholder={placeholder} style={style} ref={ref} />;
}

const ForwardInput = React.forwardRef(Input);

export default ForwardInput;
```

File `src/App.js`

```js
import React, {useRef, useEffect} from 'react';
import Input from "./components/Input";

function App() {
  const firstNameRef = useRef(null);
  const lastNameRef = useRef(null);

  useEffect(() => {
    firstNameRef.current.focus()
  })

  function firstNameKeyDown(e) {
    if(e.key === 'Enter') {
      lastNameRef.current.focus();
    }
  }

  function lastNameKeyDown(e) {
    ...
  }

  return (
    ...
      <Input
          ref={firstNameRef}
          onKeyDown={firstNameKeyDown}
          placeholder='type first name here' />
      <Input
          ref={lastNameRef}
          onKeyDown={lastNameKeyDown}
          placeholder='type last name here' />
    ...
  )
}
```

## Component LifeCycle

1. birth (mount) - page load. Method componentDidMount
2. life (update) - change in state / prop. Method componentDidUpdate
3. death (unmount). Method componentWillUnmount

## useEffect hock

File `src/App.js`

```js
import React, {useState, useEffect} from 'react;

let born = false;

function App() {
  const [growth, setGrowth] = useState(0);
  const [nirvana, setNirvana] = useState(false)

  useEffect(() => {
    if(born) {
      document.title = 'nirvana atteined';
    }
  }, [nirvana])

  // run only one time
  useEffect(() => {
    console.log('I am born')
  }, [])

  // run every updates
  useEffect(() => {
    if(born) {
      console.log('Make mistake and learn')
    } else {
      born = true
    }

    if(growth>70) {
      setNirvana(true);
    }

    return function cleanup() {
      console.log('clean up after mistake')
    }
  })

  function growHandle() {
    setGrowth(growth + 10)
  }

  return (
    <div className='App'>
      <header className='App-header'>
        <h2>Use Effect</h2>
        <h3>growth: {growth}</h3>
        <button onClick={growHandle}>Learn and grow</button>
      </header>
    </div>
  )

}
```

## useMemo

File `src/App.js`

```js
import React, { useState, useMemo, useCallback } from "react";
import Child from "./components/child";

import "./App.css";

function App() {
  const [i, setI] = useState(0);

  const callBackClick = useCallback(() => {
    console.log("called");
    setI((c) => c + 1);
  }, [i]);

  const memoizedChild = useMemo(() => {
    return <Child></Child>;
  }, []);

  return (
    <div className="App">
      <header className="App-header">
        <h3> Use Memo</h3>
        <h2>i: {i}</h2>
        <button onClick={callBackClick}>Increment I</button>
        <h3> normal render</h3>
        <Child></Child>
        <h3> Memo render</h3>
        {memoizedChild}
      </header>
    </div>
  );
}

export default App;
```

File `src\components\child.js`

```js
import React, { useEffect } from "react";

let renderCount = 0;
function Child() {
  useEffect(() => {
    renderCount++;
  });

  return <div>rendercount: {renderCount}</div>;
}

export default Child;
```

## Custom Hook: fetch data

File `src/App.js`

```js
import React, { useState } from "react";
import "./App.css";
import useCustomFetch from "./hooks/useCustomFetch";

function App() {
  const [url, setUrl] = useState(null);
  const [data, loading, error] = useCustomFetch(url);

  function getFollowers(e) {
    if (e.key === "Enter") {
      setUrl("https://api.github.com/users/" + e.target.value);
    }
  }
  return (
    <div className="App">
      <header className="App-header">
        <h2>
          GitID:
          <input onKeyPress={getFollowers}></input>
          {loading && url && <div>Loading ....</div>}
          {!loading && data && data.rData && data.rData.followers && (
            <div>Followers: {data.rData.followers}</div>
          )}
          {error && <div>Error: {error}</div>}
        </h2>
      </header>
    </div>
  );
}

export default App;
```

File `src/hooks/useCustomFetch`:

```js
import { useState, useEffect } from "react";

function useCustomFetch(url) {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);

  async function customeFetch(url) {
    try {
      let response = await fetch(url);
      let rData = await response.json();
      setData({ rData });
      setLoading(false);
    } catch (e) {
      setError(e);
      setLoading(false);
    }
  }

  useEffect(() => {
    setLoading(true);
    setTimeout(() => {
      if (url) {
        customeFetch(url);
      }
    }, 3000);
  }, [url]);

  return [data, loading, error];
}

export default useCustomFetch;
```

## Setting up Routers

Install:

```cmd
$ npm install react-router-dom --save
```

File: `src/App.js`:

```js
import React from "react";
import {
  BrowserRouter,
  Outlet,
  Route,
  Routes,
  useParams,
} from "react-router-dom";
import "./App.scss";

function User() {
  return (
    <div>
      <h2>User</h2>
      <Outlet />
    </div>
  );
}

function InfoName() {
  const params = useParams();
  return (
    <h3>
      Username: {params.name}
      <Outlet />
    </h3>
  );
}

function InfoSurname() {
  const params = useParams();
  return <>Surname: {params.surname}</>;
}

function App() {
  return (
    <BrowserRouter>
      <div className="App">
        <header className="App-header">
          <Routes>
            <Route index path="/" element={<h1>Welcome Home</h1>} />
            <Route path="about" element={<h1>Welcome About Page</h1>} />
            <Route path="user" element={<User />}>
              <Route path=":name" element={<InfoName />}>
                <Route path=":surname" element={<InfoSurname />}>
              </Route>
            </Route>
            <Route path="*" element={<h1>Not found</h1>} />
          </Routes>
        </header>
      </div>
    </BrowserRouter>
  );
}

export default App;
```

### Links

- [BrowserRouter](https://reactrouter.com/docs/en/v6/routers/browser-router)
- [Routes](https://reactrouter.com/docs/en/v6/components/routes)
- [Route](https://reactrouter.com/docs/en/v6/components/route)

## Route Links

File: `src/App.js`

```js
import React from "react";
import {
  ...
  NavLink,
} from "react-router-dom";

...

const activeStyle = {
  color: "red",
};

function App() {
  return (
    <BrowserRouter>
      <div className="App">
        <header className="App-header">
          <nav>
            <menu style={{ listStyle: "none" }}>
              <li>
                <NavLink
                  to="/"
                  style={({ isActive }) => (isActive ? activeStyle : undefined)}
                >
                  Home
                </NavLink>
              </li>
              <li>
                <NavLink
                  to="about"
                  style={({ isActive }) => (isActive ? activeStyle : undefined)}
                >
                  About
                </NavLink>
              </li>
            </menu>
          </nav>
          <Routes>
            <Route index path="/" element={<h1>Welcome Home</h1>} />
            <Route path="about" element={<h1>Welcome About Page</h1>} />
            <Route path="user/" element={<User />}>
              <Route path=":name" element={<InfoName />}>
                <Route path=":surname" element={<InfoSurname />} />
              </Route>
            </Route>
            <Route path="*" element={<h1>Not found</h1>} />
          </Routes>
        </header>
      </div>
    </BrowserRouter>
  );
}

export default App;
```

### Links:

- [NavLink](https://reactrouter.com/docs/en/v6/components/nav-link)
- [Link](https://reactrouter.com/docs/en/v6/components/link)

## Route redirect

Example: redirect to main page if logout

```js
import React, { useState } from "react";
import {
  BrowserRouter,
  Outlet,
  Route,
  Routes,
  NavLink,
  Navigate,
  useParams,
} from "react-router-dom";
import PropTypes from "prop-types";
import "./App.scss";

function User() {
  return (
    <div>
      <h2>User</h2>
      <Outlet />
    </div>
  );
}

function InfoName({ loggedIn }) {
  const params = useParams();
  return loggedIn ? (
    <h3>
      Username: {params.name}
      <br /> <Outlet />
    </h3>
  ) : (
    <Navigate to="/" replace={true} />
  );
}

InfoName.propTypes = {
  loggedIn: PropTypes.bool,
};

function InfoSurname() {
  const params = useParams();
  return <>Surname: {params.surname}</>;
}

const activeStyle = {
  color: "red",
};

function App() {
  const [loggedIn, setLoggedIn] = useState(false);
  const onClickHandle = () => {
    setLoggedIn(!loggedIn);
  };
  return (
    <BrowserRouter>
      <div className="App">
        <header className="App-header">
          <nav>
            <menu style={{ listStyle: "none" }}>
              <li>
                <NavLink
                  to="/"
                  style={({ isActive }) => (isActive ? activeStyle : undefined)}
                >
                  Home
                </NavLink>
              </li>
              <li>
                <NavLink
                  to="about"
                  style={({ isActive }) => (isActive ? activeStyle : undefined)}
                >
                  About
                </NavLink>
              </li>
              <li>
                <NavLink
                  to="user/john/doe"
                  style={({ isActive }) => (isActive ? activeStyle : undefined)}
                >
                  User John Doe
                </NavLink>
              </li>
            </menu>
          </nav>
          <button onClick={onClickHandle}>
            {loggedIn ? "logout" : "login"}
          </button>
          <Routes>
            <Route index path="/" element={<h1>Welcome Home</h1>} />
            <Route path="about" element={<h1>Welcome About Page</h1>} />
            <Route path="user/" element={<User />}>
              <Route path=":name" element={<InfoName loggedIn={loggedIn} />}>
                <Route path=":surname" element={<InfoSurname />} />
              </Route>
            </Route>
            <Route path="*" element={<h1>Not found</h1>} />
          </Routes>
        </header>
      </div>
    </BrowserRouter>
  );
}

export default App;
```

## useContext Hook

Share information between different pages

File: `src/contexts/messageContext.js`:

```js
import { createContext } from "react";

const messageContext = createContext(null);

export default messageContext;
```

File: `src/pages/AboutPage.js`:

```js
import { useContext } from "react";
import messageContext from "../contexts/messageContext";

function AboutPage() {
  const [message, setMessage] = useContext(messageContext);
  return (
    <>
      <h1>Welcome About Page</h1>
      <h2>Message: {message}</h2>
      <button onClick={() => setMessage("New message")}>Change message</button>
    </>
  );
}

export default AboutPage;
```

File `src/App.js`:

```js
import React, { useState } from "react";
import { BrowserRouter, Route, Routes } from "react-router-dom";
import messageContext from "./contexts/messageContext";
import AboutPage from "./pages/AboutPage";
import "./App.scss";

function App() {
  const [message, setMessage] = useState("I am being share");
  return (
    <BrowserRouter>
      <messageContext.Provider value={[message, setMessage]}>
        <div className="App">
          <header className="App-header">
            ...
            <Routes>
              <Route index path="/" element={<h1>Welcome Home</h1>} />
              <Route path="about" element={<AboutPage />} />
              ...
            </Routes>
          </header>
        </div>
      </messageContext.Provider>
    </BrowserRouter>
  );
}

export default App;
```

## React + Redux

Install dependencies:

```cmd
$ npm install react-router-dom redux react-redux
```

Add file `src/store/balanceReducer.js`:

```JavaScript
const initialState = {
  balance: 0,
};

function reducer(state = initialState, action) {
  switch (action.type) {
    case "balance/deposit":
      return {
        balance: state.balance + action.payload,
      };
    case "balance/withdraw":
      return {
        balance: state.balance - action.payload,
      };
    default:
      return state;
  }
}

export default reducer;
```

Edit file: `src/index.js`:

```JavaScript
...
import reducer from "./store/balanceReducer";
import { Provider } from "react-redux";
import { createStore } from "redux";

const store = createStore(reducer);

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
);
...
```

Add file: `src/page/homePage.js`:

```JavaScript
import { useSelector } from "react-redux";

function HomePage() {
  const balance = useSelector((state) => state.balance);
  return <h1>Balance: {balance}</h1>;
}

export default HomePage;
```

Add file: `src/page/depositPage.js`:

```JavaScript
import { useSelector, useDispatch } from "react-redux";

function DepositPage() {
  const balance = useSelector((state) => state.balance);
  const dispatch = useDispatch();
  function onDepositHandle() {
    dispatch({ type: "balance/deposit", payload: 10 });
  }

  return (
    <>
      <h1>Balance: {balance}</h1>
      <button onClick={onDepositHandle}>Deposit</button>
    </>
  );
}

export default DepositPage;
```

Add file: `src/page/withdrwawPage.js`:

```JavaScript
import { useSelector, useDispatch } from "react-redux";

function WithdrawPage() {
  const balance = useSelector((state) => state.balance);
  const dispatch = useDispatch();
  function onWithdrawHandler() {
    dispatch({ type: "balance/withdraw", payload: 10 });
  }
  return (
    <>
      <h1>Balance: {balance}</h1>
      <button onClick={onWithdrawHandler}>Withdraw</button>
    </>
  );
}

export default WithdrawPage;
```

Edit `src/App.js` file:

```JavaScript
import { BrowserRouter, Route, Routes, NavLink } from "react-router-dom";
import "./App.scss";
import DepositPage from "./pages/depositPage";
import HomePage from "./pages/homePage";
import WithdrawPage from "./pages/withdrawPage";

const activeStyle = {
  color: "red",
};

function App() {
  return (
    <BrowserRouter>
      <div className="App">
        <header className="App-header">
          <nav>
            <menu style={{ listStyle: "none" }}>
              <li>
                <NavLink
                  to="/"
                  style={({ isActive }) => (isActive ? activeStyle : undefined)}
                >
                  Home
                </NavLink>
                <NavLink
                  to="deposit"
                  style={({ isActive }) => (isActive ? activeStyle : undefined)}
                >
                  Deposit
                </NavLink>
                <NavLink
                  to="withdraw"
                  style={({ isActive }) => (isActive ? activeStyle : undefined)}
                >
                  Withdraw
                </NavLink>
              </li>
            </menu>
          </nav>
          <Routes>
            <Route path="/" element={<HomePage />} />
            <Route path="deposit" element={<DepositPage />} />
            <Route path="withdraw" element={<WithdrawPage />} />
          </Routes>
        </header>
      </div>
    </BrowserRouter>
  );
}

export default App;
```

## Combining multiple reducers

See previous code.

Modify file: `src/pages/homePage.js`

```js
import { useSelector, useDispatch } from "react-redux";

function HomePage() {
  const balance = useSelector((state) => state.balanceReducer.balance);
  const loan = useSelector((state) => state.loanReducer.loan);
  const dispatch = useDispatch();
  function applyLoanHandle() {
    dispatch({ type: "LOAN" });
  }
  return (
    <div>
      <h1>Balance: {balance}</h1>
      <h2>Loan: {loan ? "Applied" : "Loan needed"}</h2>
      <button onClick={applyLoanHandle}>Apply for loan</button>
    </div>
  );
}

export default HomePage;
```

Add file `src/reducers/loanReducer.js`:

```js
const initialState = {
  loan: false,
};
function loanReducer(state = initialState, action) {
  switch (action.type) {
    case "LOAN":
      return { loan: true };
    default:
      return state;
  }
}

export default loanReducer;
```

Modify file `src/index.js`:

```js
...
import { createStore, combineReducers } from "redux";
import balanceReducer from "./store/balanceReducer";
import loanReducer from "./store/loanReducer";

const store = createStore(
  combineReducers({
    balanceReducer,
    loanReducer,
  })
);

....
```

Modify files: `src/pages/homePage.js`, `src/pages/depositPage.js`, `src/pages/withdrawPage.js`:

```js
...
function ...
  const balance = useSelector((state) => state.balanceReducer.balance);
...
```

## Redux Thunk

Thunk middleware for Redux. It allows writing functions with logic inside that 
can interact with a Redux store's dispatch and getState methods.

Install dependencies:

```cmd
npm install redux-thunk --save
```

Modify `src/index.js`:

```js
...
import { createStore, combineReducers, applyMiddleware } from "redux";

import balanceReducer from "./store/balanceReducer";
import loanReducer from "./store/loanReducer";
import thunk from "redux-thunk";

const store = createStore(
  combineReducers({
    balanceReducer,
    loanReducer,
  }),
  applyMiddleware(thunk)
);
...
```

Add action file `src/actions/balanceActions.js`:

```js
import { DEPOSIT, LOADING, WITHDRAW } from "../constants";

export function loading() {
  return { type: LOADING };
}

export function deposit(amount) {
  return {
    type: DEPOSIT,
    payload: amount,
  };
}

export function depositAsync(amount) {
  return (dispatch) => {
    dispatch(loading());
    setTimeout(() => {
      dispatch(deposit(amount));
    }, 5000);
  };
}

export function withdraw(amount) {
  return {
    type: WITHDRAW,
    payload: amount,
  };
}
```

Add constants to file `src/constants.js`:

```js
export const DEPOSIT = "DEPOSIT";

export const WITHDRAW = "WITHDRAW";

export const LOADING = "LOADING";
```

Modify file `src/pages/depositPage.js`:

Add file `src/page/counterPage.js`:

```js
import { useSelector, useDispatch } from "react-redux";
import { depositAsync } from "../actions/balanceActions";

function DepositPage() {
  const balance = useSelector((state) => state.balanceReducer.balance);
  const loading = useSelector((state) => state.balanceReducer.loading);
  const dispatch = useDispatch();

  function onDepositHandle() {
    dispatch(depositAsync(10));
  }

  return (
    <>
      {loading ? <h1>Loading...</h1> : <h1>Balance: {balance}</h1>}

      <button onClick={onDepositHandle}>Deposit</button>
    </>
  );
}

export default DepositPage;
```

## useReducer Hook

Hook works as redux but for local state:

```js
import { useReducer } from "react";

const initialState = {
  count: 0,
};

function reducerFunction(state, action) {
  switch (action.type) {
    case "INCREMENT":
      return { count: state.count + 1 };
    case "DECREMENT":
      return { count: state.count - 1 };
    default:
      return state;
  }
}

function CounterPage() {
  const [state, dispatch] = useReducer(reducerFunction, initialState);
  function plusOneHandler() {
    dispatch({ type: "INCREMENT" });
  }
  function minusOneHandler() {
    dispatch({ type: "DECREMENT" });
  }
  return (
    <div>
      <h1>useRedux Hook Demo</h1>
      <h3>Count: {state.count}</h3>
      <button onClick={plusOneHandler}>Plus One</button>
      <button onClick={minusOneHandler}>Minus One</button>
    </div>
  );
}

export default CounterPage;
```

## using Redux Tookit

Install dependencies:

```cmd
npm install @reduxjs/toolkit react-redux
```

Add file `src/store/slice.js`:

```JavaScript
import { createSlice } from "@reduxjs/toolkit";

const balanceSlice = createSlice({
  name: "balance",
  initialState: {
    balance: 0,
  },
  reducers: {
    deposit: (state, action) => {
      state.balance += action.payload;
    },
    withdraw: (state, action) => {
      state.balance -= action.payload;
    },
  },
});

export const { deposit, withdraw } = balanceSlice.actions;
export default balanceSlice.reducer;
```

Modify `src/index.js` file:

```JavaScript
...
import { Provider } from "react-redux";

import balanceReducer from "./store/slice";
import { configureStore } from "@reduxjs/toolkit";

const store = configureStore({
  reducer: balanceReducer,
});

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
);
...
```

Modify `src/pages/depositPage.js` file:

```JavaScript
import { useSelector, useDispatch } from "react-redux";
import { deposit } from "../store/slice";

function DepositPage() {
  const balance = useSelector((state) => state.balance);
  const dispatch = useDispatch();

  function onDepositHandle() {
    dispatch(deposit(10));
  }

  return (
    <>
      <h1>Balance: {balance}</h1>
      <button onClick={onDepositHandle}>Deposit</button>
    </>
  );
}

export default DepositPage;
```

## PropTypes

Demo component `src/components/MyComp.js`:

```js
import PropTypes from "prop-types";

function MyComp({ str, onClick, obj }) {
  return (
    <div onClick={onClick}>
      {str} {obj.name}
    </div>
  );
}

MyComp.propTypes = {
  str: PropTypes.string,
  onClick: PropTypes.func.isRequired,
  obj: PropTypes.shape({
    age: PropTypes.number,
    name: PropTypes.string,
    gender: PropTypes.oneOf(["M", "F"]),
    birthdate: PropTypes.instanceOf(Date),
  }),
};

MyComp.defaultProps = {
  str: "Hello world",
};

export default MyComp;
```

## TypeScript
