# code-assistant [![version](https://img.shields.io/github/release/hchiam/code-assistant)](https://github.com/hchiam/code-assistant/releases)

## notes

- goals/tests
- possible actions
  - what actions to limit AI to be able to do, for safety?
- state
- read ideas
  - google search for code?
  - google search plain prose for ideas?
  - google search latest specs?
  - google search latest code libraries?
- reason/ideate
  - some logic to come up with ideas from possible actions
  - some logic to come up with ideas from what read

## re-summary

- state, actions + (read? ideate?), test
- state, tests, actions, reading, ideation
  - reading --> more actions
  - ideation --> choosing actions + combining actions

## first draft

- state = comes from browser
  - EASY, I JUST NEED TO DECIDE WHAT'S ALLOWED TO BE ACCESSED
  - EASY, I NEED TO DECIDE WHAT + HOW MUCH EXTRA VARS TO ALLOW
- tests = comes from user
  - EASY, I JUST NEED TO DEFINE THE TESTS FOR IT TO RUN
- actions = allow-list of functions
  - EASY, I JUST NEED TO DECIDE WHICH ONES
  - HARD: THINK OF WHICH ONES TO ALLOW, OR GIVE PERMISSION BASED ON WHAT READING SOURCES IT FOUND
- reading = google / sourceFetch-server
  - HARD: THINK HOW TRANSLATE SAFELY INTO ACTIONS TO COMBINE
- ideation = allow-list of ways & depth of combining actions
  - EASY: THINK OF REALISTIC/USEFUL WAYS TO COMBINE FUNCTIONS
  - HARD: IMPLEMENT CODE TO GENERATE CODE

## run-through conceptual examples

<details>
<summary>simple example to get started</summary>

- state
  - window
  - outputValue
  - outputObject
- tests: get the window width
  - outputValue === window.innerWidth
  - outputObject === window
- actions

- get actions by searching the input window object API of props + functions for things that most closely match width

  ```js
  // reference https://github.com/hchiam/code-explorer/blob/master/public/api-search.js
  function findKeys(obj, lookingFor) {
    const keys = new Set();
    Object.keys(obj).map((k) => {
      keys.add(k);
    });
    Object.keys(Object.getPrototypeOf(obj)).map((k) => {
      keys.add(k);
    });
    if (obj.__proto__ && Object.getPrototypeOf(obj.__proto__)) {
      Object.getOwnPropertyNames(Object.getPrototypeOf(obj.__proto__)).map(
        (k) => {
          keys.add(k);
        }
      );
    }
    Object.getOwnPropertyNames(obj).map((k) => {
      keys.add(k);
    });
    const matches = [];
    const found = [...keys].some((k) => {
      const found = k.toLowerCase().includes(lookingFor.toLowerCase());
      if (found) matches.push(k);
    });
    return matches;
  }
  console.log(findKeys(Math, "pow"));
  console.log(findKeys(NaN, "string"));
  // TODO: get google description of object.key
  // TODO: show user the info and ask user permission
  // TODO: add to list of allowed actions
  // TODO: perform actions that were allowed
  // TODO: always ask about actions that weren't allowed
  ```

- reading
  - search google / sourceFetch-server
    - <https://github.com/hchiam/code-explorer/blob/master/public/google.js>
    - <https://codepen.io/hchiam/full/PEMgBN>
  - search MDN web APIs
    - TODO: get snippets from MDN
  - (what about frameworks or libraries?)
    - (just stick to low-level APIs for now?)
- ideation

  - combos: object, function(), object.object, object.function(), function(object), object.function(object2), function(object).function(), any of the above but in sequence, any of the above, but inside a function callback

    - but object.function() is like object.object but typeof
    - and function() is actually window.function()
    - and try function(object) only if has param

      ```js
      function hasParameters(fun) {
        const match = fun.toString().match(/\((.+?)\)/)?.[1];
        if (!match) return false;
        return Array.from(match).some((x) => x !== " ");
      }
      // TODO: test a = () => {}
      // TODO: test a = ( ) => {}
      // TODO: test a = (b) => {}
      // TODO: test a = (b,c) => {}
      // TODO: test a = (b, c) => {}
      // TODO: test function a() {}
      // TODO: test function a( ) {}
      // TODO: test function a(b) {}
      // TODO: test function a(b,c) {}
      // TODO: test function a(b, c) {}
      ```

    - so more like:
      - 1. object === answer? or test passes?
      - 2. window === answer? or test passes?
      - 3. object.somePropOrFun === answer? or test passes?
      - 4. object.somePropOrFun() === answer? or test passes?
      - 5. window.somePropOrFun === answer? or test passes?
      - 6. window.somePropOrFun() === answer? or test passes?
      - 7. object.someFun(someObj) === answer? or test passes?
        - if hasParameters(object.someFun)
      - 8. window.someFun(someObj) === answer? or test passes?
        - if hasParameters(window.someFun)
      - a) sequence --> test passes?
      - b) chained === answer? or test passes?
      - c) nested === answer? or test passes?

  ```js
  function exploreApi_1_step(obj, key) {
    return obj; // 1
    return window; // 2
    if (findKeys(obj, key)) {
      return obj.key; // 3
      if (typeof obj.key === "function") {
        return obj[key](); // 4
        if (hasParameters(obj.key)) {
          return obj[key](`someOtherObject???`); // 5
        }
      }
    }
    if (findKeys(window, key)) {
      return window.key; // 6
      if (typeof window.key === "function") {
        return window[key](); // 7
        if (hasParameters(window.key)) {
          return window[key](`someOtherObject???`); // 8
        }
      }
    }
  }
  function exploreApi_2_step(objectKeyPairsOrActionsAvail) {
    const actions = objectKeyPairsOrActionsAvail;
    for (let i = 0; i < actions[i].length; i++) {
      for (let j = 0; j < actions[j].length; j++) {
        trySequence(actions[i], actions[j]); // a)
        tryChain(actions[i], actions[j]); // b)
        tryNesting(actions[i], actions[j]); // c)
      }
    }
  }
  function trySequence(action1, action2) {
    return output;
  }
  function tryChain(action1, action2) {
    return output;
  }
  function tryNesting(action1, action2) {
    return output;
  }
  ```

</details>

<hr/>

<details>
<summary>harder example to get more useful</summary>

- state
  - document.body
  - outputValue = []
  - outputObject = outputValue
- tests: get the URLs of all the images on the page
  - outputValue === ['1.png','2.png','3.png']
  - outputObject === outputValue
    - maybe we usually don't need to validate parent object?
- actions
  - get actions by searching the document.body object API of props + functions for things that most closely match: "get", "URLs", "all", "images", "page"
    - (see JS code in previous example)
- reading
  - search google / sourceFetch-server
    - <https://github.com/hchiam/code-explorer/blob/master/public/google.js>
    - <https://codepen.io/hchiam/full/PEMgBN>
  - search MDN web APIs
    - TODO: get snippets from MDN
  - (what about frameworks or libraries?)
    - (just stick to low-level APIs for now?)
- ideation
  - (see JS code in previous example)

</details>

## TODO

- TODO: add getting code snippets from codegrepper
- TODO: try adding snippets from MDN to the sourceFetch-server
- TODO: flesh out more of the 2 ideation code snippets

## earlier idea

<https://github.com/hchiam/code-explorer>
