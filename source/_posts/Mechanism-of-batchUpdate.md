---
title: Mechanism of batchUpdate
abbrlink: 43124a1
date: 2020-04-06 15:08:22
categories:
    - React
---

Update your component's state by either `useState` or `this.setState`, this will make your component re-render. But there is a little different when you have variable calls to update the state.

{% note info %}
Currently (React 16 and earlier), only updates inside React event handlers are batched by default. There is an unstable API to force batching outside of event handlers for rare cases when you need it. In future versions (probably React 17 and later), React will batch all updates by default so you won't have to think about this.

---
the follow two points are conclusion by myself, it may not be accurate enough
- **make update calls in a sync way**: React will make the updates in a batch, instead of one at a time, reducing the number of render the component will make.
- **make update calls in a async way**: React makes the updates one by one, instead of in a batch, this means you'll get multiple re-renders.
{% endnote %}

There is a example, [example in codesandbox](https://codesandbox.io/s/mechanism-of-batchupdate-s97f9)

```js
// entry
import React from "react";
import ReactDOM from "react-dom";
import ClassComponent from "./component/ClassComponent";
import FunctionComponent from "./component/FunctionComponent";
import "./styles.css";

function App() {
  return (
    <div className="App">
      <div className="component">
        <ClassComponent />
      </div>
      <div className="component">
        <FunctionComponent />
      </div>
    </div>
  );
}

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);
```

```js
// FunctionComponent
import React, { useState, useRef } from "react";

const FunctionComponent = () => {
  const [firstState, setFirstState] = useState(0);
  const [secondState, setSecondState] = useState(0);

  const syncAdd = () => {
    setFirstState(firstState + 1);
    setSecondState(secondState + 1);
  };

  const asyncAdd = () => {
    setTimeout(() => {
      setFirstState(firstState + 1);
      setSecondState(secondState + 1);
    }, 0);
  };

  const renderTimes = useRef(0);
  console.log(
    `Function Component 第 ${++renderTimes.current} 次 render, firstState: ${firstState}, secondState: ${secondState}`
  );

  return (
    <>
      <div>Function Component</div>
      <button onClick={syncAdd}>syncAdd</button>
      <button onClick={asyncAdd}>asyncAdd</button>
    </>
  );
};

export default FunctionComponent;
```

```js
// ClassComponent
import React, { Component } from "react";

class ClassComponent extends Component {
  constructor(props) {
    super(props);
    this.state = {
      firstState: 0,
      secondState: 0
    };
    this.renderTimes = 0;
  }

  syncAdd = () => {
    const { firstState, secondState } = this.state;
    this.setState({ firstState: firstState + 1 });
    this.setState({ secondState: secondState + 1 });
  };

  asyncAdd = () => {
    setTimeout(() => {
      const { firstState, secondState } = this.state;
      this.setState({ firstState: firstState + 1 });
      this.setState({ secondState: secondState + 1 });
    });
  };

  render() {
    const { firstState, secondState } = this.state;
    console.log(
      `Class Component 第 ${++this
        .renderTimes} 次 render, firstState: ${firstState}, secondState: ${secondState}`
    );
    return (
      <>
        <div>Class Component</div>
        <button onClick={this.syncAdd} onMouseUp={this.methodAfterSyncAdd}>
          syncAdd
        </button>
        <button onClick={this.asyncAdd} onMouseUp={this.methodAfterAsyncAdd}>
          asyncAdd
        </button>
      </>
    );
  }
}

export default ClassComponent;
```

References
- [Simplifying state management in React apps with batched updates](https://blog.logrocket.com/simplifying-state-management-in-react-apps-with-batched-updates/)
- [Does React keep the order for state updates](https://stackoverflow.com/questions/48563650/does-react-keep-the-order-for-state-updates/48610973#48610973)
