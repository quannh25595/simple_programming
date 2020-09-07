---
layout: 'post'
title: "Handle API calling process with custom React hooks"
categories: react hooks
permalink: /:categories/:title
description: A react hooks approach for a classic problem
---

React is a popular UI library nowadays. With the debut of hooks, React component now is much cleaner and the logic is more reuseable.

One of the common cases with React is when we try to perform some API calling and tracking it's state.

<img src="{{'/public/img/2020-09-05-demo-async.gif' | absolute_url}}" alt="demo-async"/>

## The traditional way

So this is a common & traditional way we deal with API calling process

```jsx
import React, { useEffect, useState } from "react";

export const ListUser = () => {
  const [loading, setLoading] = useState(false);
  const [results, setResults] = useState([]);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true);
    fetch("https://randomuser.me/api/?results=500")
      .then((res) => res.json())
      .then((res) => {
        setResults(res.results);
        setError(null);
      })
      .catch((e) => {
        setError(e);
      })
      .finally(() => {
        setLoading(false);
      });
  }, []);

  if (loading) return <div>Loading</div>;
  if (error) return <div>{error.message}</div>;

  return (
    <div>
      {results.map((item) => (
        <div key={item.id.value}>{item.name.first}</div>
      ))}
    </div>
  );
}
```

What we have basically are:

* `loading`: A state for indicating if the data is fetching or not
* `results`: A state that holds the data from response
* `error`: A state for storing the error if something went wrong

With these states, we can basically tracking the API calling process, as you can see in the gif above

What if there are more API calls inside our component? Things become messy. We'll need more & more states for specific API. For example:

```jsx
...

const [loading_1, setLoading_1] = useState(false);
const [results_1, setResults_1] = useState([]);
const [error_1, setError_1] = useState(null);

const [loading_2, setLoading_2] = useState(false);
const [results_2, setResults_2] = useState([]);
const [error_2, setError_2] = useState(null);

...
```

We can see that we got some duplications in processing here: every API call will need the `loading`, `result` and `error` state. It would be nice if we can somehow extract them and reuse it wherever we need.
This is the place that React custom hooks shining.

## Custom hooks

>You should definitely checkout the tutorial from the official documentation [here](https://reactjs.org/docs/hooks-custom.html)

We need to understand the custom hooks first. Look at the gif below

<img src="{{'/public/img/2020-09-05-word-count.gif' | absolute_url}}" alt="word-count"/>

As you can see from the image, we have a quite simple program: counting the words inside the `textarea`. The code should look like:

{% raw %}

```jsx
import React, { useState, useEffect } from "react";

export const WordCount = () => {
  const [value, setValue] = useState("");
  const [wordCount, setWordCount] = useState(0);

  // use useEffect to automatically recalculate the wordCount whenever the value changed
  useEffect(() => {
    setWordCount(value.trim() ? value.split(" ").length : 0);
  }, [value]);

  return (
    <div>
      <textarea
        style={{ width: "100%", height: 200 }}
        value={value}
        onChange={(event) => setValue(event.target.value)}
      />
      <div style={{ display: "flex", justifyContent: "space-between" }}>
        <button onClick={() => setValue("")}>Clear</button>
        <span>Word Count: {wordCount}</span>
      </div>
    </div>
  );
};

```

{% endraw %}

What if we also need the `wordCount` behavior for other components as well? Can we reuse the `wordCount` logic somewhere else?

Definitely yes 🙂. Let's extract the `wordCount` logic to a custom hooks. Look at the **useWordCount.js** file:

```jsx
import { useState, useEffect } from "react";

export const useWordCount = (textValue) => {
  const [count, setCount] = useState(0);

  // update the count state whenever textValue changed
  useEffect(() => {
    setCount(textValue.trim() ? textValue.split(" ").length : 0);
  }, [textValue]);

  return count;
};
```

Now we have the `wordCount` logic separated. And basically we can count any text we want. Get back to our component, and add few more elements that also need to be counted. Our old `WordCount` component should look like this:

{% raw %}

```jsx
import React, { useState } from "react";
import { useWordCount } from "./useWordCount";

export const WordCount = () => {
  const [textAreaValue, setTextAreaValue] = useState("");
  const [inputValue, setInputValue] = useState("");

  // Here is the count value we get from our custom hooks
  const textAreaCount = useWordCount(textAreaValue);
  const inputCount = useWordCount(inputValue);

  return (
    <div>
      <textarea
        style={{ width: "100%", height: 200 }}
        value={textAreaValue}
        onChange={(event) => setTextAreaValue(event.target.value)}
      />
      <div style={{ display: "flex", justifyContent: "space-between" }}>
        <button onClick={() => setTextAreaValue("")}>Clear</button>
        <span>Word Count: {textAreaCount}</span>
      </div>
      <div style={{ marginTop: 10 }}>
        <input
          type="text"
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
        />
        <span>Word Count: {inputCount}</span>
      </div>
    </div>
  );
};

```

{% endraw %}

Great! Here is the result:

<img src="{{'/public/img/2020-09-05-word-count-with-hooks.gif' | absolute_url}}" alt="word-count-with-hooks"/>

As you can see, we have a cleaner code, and a reuseable hooks that can be used in other components.

## Custom hooks for API Calling Problem

With the idea using custom hooks for logic separation, I wrote a npm package to simplify the API calling process. Check it out at [https://www.npmjs.com/package/react-hook-async](https://www.npmjs.com/package/react-hook-async)

### Installation

```bash
npm i react-hook-async
```

or

```bash
yarn add react-hook-async
```

### Usage

#### Basic

The basic scenario is when you're trying to perform an API calling inside your React component:

```jsx
import React, {useEffect} from 'react'
import {useAsync} from 'react-hook-async'

const fetchUsers = () =>
    fetch('https://randomuser.me/api/?results=50')
    .then((res) =>
      res.json()
    );

export const ListUser = () => {
  const [apiData, executeFetchUsers] = useAsync([], fetchUsers)

  useEffect(() => {
    executeFetchUsers()
  }, [executeFetchUsers])

  const {loading, result, error} = apiData;

  if (loading) return <div>Loading</div>;
  if (error) return <div>{error.message}</div>;

  return (
    <div>
      {result.map((item) => (
        <div key={item.id.value}>{item.name.first}</div>
      ))}
    </div>
  );
}
```

We've got rid off 3-4 states inside our component, which is cool thing, isn't it? 😎

Some explanations:

* The `useAsync` returns an array:
  * The first element in array is an object that holds all states of API calling process, includes `result`, `error`, `loading` and `lastFetch`. Without this `useAsync`, we'll need to store them as local state.
  * The second element in returned array is a function, used for actually perform an API call.

  Basically, you could think about somethings that similar to `useState` hook, which also returns an array with similar idea: a state variable and a function to change this state.
* `useAsync` takes 2 args:
  * The first arg is the default value for `result` field. You can pass any values that you want. Here we use an empty array to avoid type checking: you no longer need to check if `result` is an array
  * The second arg is actually a function that returns a promise. You have to make sure that the function will return a promise, because the promise will be used inside `useAsync` hooks.

As you can see from the example above, now we can simply perform API calling by using the function that returns by `useAsync`, without the need to use internal state to tracking it.

#### Passing parameters

The good things is that the "execute" function is also able to receive parameters, and they would be passed to your API calling function. Let's look at the below code:

```jsx
...
const fetchUsers = (numOfUser) =>
    fetch(`https://randomuser.me/api/?results=${numOfUser}`)
    .then((res) =>
      res.json()
    );

export const ListUser = () => {
  const [apiData, executeFetchUsers] = useAsync([], fetchUsers)

  useEffect(() => {
    executeFetchUsers(50)
  }, [executeFetchUsers])

  const {loading, result, error} = apiData;
  ...
}
```

With this ability, the API calling function could be customized, just like what we're expecting.

#### Chaining API calling and error handling

There would be the case that we want to perform an API call after another, or just simply do some actions after the calling process is completed/crashed. The `execute` function actually returns a Promise, allows you to further resolve it, or handle error happened from inside. Let's see another example:

```jsx
...
const fetchUsers = (numOfUser) =>
    fetch(`https://randomuser.me/api/?results=${numOfUser}`)
    .then((res) =>
      res.json()
    );

const fetchFirstUser = (id) =>
    fetch(`https://randomuser.me/api/?id=${id}`)
    .then((res) =>
      res.json()
    );

export const ListUser = () => {
  const [apiData, executeFetchUsers] = useAsync([], fetchUsers)
  const [firstUserApiData, executeFetchFirstUser] = useAsync(
    [],
    fetchFirstUser
  )

  useEffect(() => {
    executeFetchUsers(50)
      .then(users => executeFetchFirstUser(users[0].id))
      .catch(err => {
        console.log('Something went wrong:', err)
      })
  }, [executeFetchUsers, executeFetchFirstUser])

  const {loading, result, error} = apiData;
  ...
}
```

### Downside

The only problem so far is, well, you'll need to pass the `execute` function to the deps array of `useEffect` or `useCallback`, although I'm pretty sure that it would never be changed. You probably could visit the repo [here](https://github.com/quannh25595/react-hook-async) and give it a try. Any PRs are warmly welcomed 🙂

## Conclusion

With React hook, and ability to create your own custom hook. Your codebase would be much cleaner and easier to read. A lot of libraries has updated to a simpler approach with hooks. You definitely should check it out.

## See also

* [https://reactjs.org/docs/hooks-intro.html](https://reactjs.org/docs/hooks-intro.html)
* [https://www.youtube.com/watch?v=dpw9EHDh2bM](https://www.youtube.com/watch?v=dpw9EHDh2bM)