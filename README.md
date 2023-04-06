# useMemoCompare

This code defines a custom React hook called useMemoCompare, which is used in the MyComponent function to compare the previous and current values of an object and return the previous object reference if they are equal based on a custom comparison function.

The MyComponent function takes an object as a prop and initializes a state variable called state using the useState hook. It then uses useMemoCompare to create a new variable called objFinal, which will either be the same as the obj prop or the previous value of objFinal if the id property of obj hasn't changed. This is done to prevent unnecessary re-renders and avoid infinite loops caused by state updates triggering the effect to run again.

The useEffect hook is then used to run an effect whenever objFinal changes. Inside the effect, a method is called on the objFinal object and the result is used to update the state variable using the setState function.

The useMemoCompare function takes two arguments: next, which is the current value to compare, and compare, which is a custom comparison function that takes the previous and next values as arguments and returns a boolean indicating whether they are equal. The function stores the previous value using the useRef hook and checks whether the previous and next values are equal by calling the comparison function. If they are equal, it returns the previous value stored in the previousRef ref using the ternary operator. If they are not equal, it updates the previousRef ref with the new value and returns the new value.

Overall, the purpose of this code is to allow developers to compare objects and return the previous reference if they are equal, in order to avoid unnecessary re-renders and potential infinite loops caused by state updates triggering effects that cause further updates.

```javascript
import React, { useState, useEffect, useRef, useCallback } from "react";

// Usage
function MyComponent({ obj }) {
  const [state, setState] = useState();
  
  // Use the previous obj value if the "id" property hasn't changed
  const objFinal = useMemoCompare(obj, (prev, next) => {
    return prev && prev.id === next.id;
  });
  
  // Here we want to fire off an effect if objFinal changes.
  // If we had used obj directly without the above hook and obj was technically a
  // new object on every render then the effect would fire on every render.
  // Worse yet, if our effect triggered a state change it could cause an endless loop
  // where effect runs -> state change causes rerender -> effect runs -> etc ...
  useEffect(() => {
    // Call a method on the object and set results to state
    const someMethod = async () => {
      const value = await objFinal.someMethod();
      setState(value);
    };
    someMethod();
  }, [objFinal]);
  
  return <div> ... </div>;
}

// Custom hook to memoize and compare objects
function useMemoCompare(next, compare) {
  // Ref for storing previous value
  const previousRef = useRef();
  // Use a callback to create a memoized comparison function
  const isEqual = useCallback(
    compare(previousRef.current, next),
    [compare, next]
  );
  
  // If not equal update previousRef to next value.
  // We only update if not equal so that this hook continues to return
  // the same old value if compare keeps returning true.
  useEffect(() => {
    if (!isEqual) {
      previousRef.current = next;
    }
  });
  
  // Finally, if equal then return the previous value
  return isEqual ? previousRef.current : next;
}

```
