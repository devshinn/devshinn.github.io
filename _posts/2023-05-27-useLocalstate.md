---
title: Using localStorage with react custom hooks
excerpt: useLocalStorage
tags:
  - react
  - hook
  - localstorage
  - sessionstorage
categories:
  - react/next
toc_sticky: true
published: true
sitemap:
  changefreq: daily
  priority: 1.0
---

![image1](/assets/images/posts/1685177709946.png)

### 사용 예시

```ts
const HooksTestPage = () => {
  const [store, setStore] = useLocalStorage<Boolean>("isReact", true);

  const onChangeStoreValue = () => {
    setStore((v: boolean) => (v ? false : true));
  };

  return (
    <div className="p-5 m-5 border-2 rounded">
      <div>storeValue : {store ? "true" : "false"}</div>
      <br />
      <button onClick={onChangeStoreValue}>changeValue</button>
    </div>
  );
};

export default HooksTestPage;
```

### custom hook

```ts
// Hook

function useLocalStorage<T>(
  key: string,
  initialValue: T
): [value: T, setStore: React.Dispatch<T | any>] {
  // State to store our value
  // Pass initial state function to useState so logic is only executed once
  const [storedValue, setStoredValue] = useState(() => {
    if (typeof window === "undefined") {
      return initialValue;
    }
    try {
      // Get from local storage by key
      const item = window.localStorage.getItem(key);
      // Parse stored json or if none return initialValue
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      // If error also return initialValue
      console.log(error);
      return initialValue;
    }
  });
  // Return a wrapped version of useState's setter function that ...
  // ... persists the new value to localStorage.
  const setValue = (value: any) => {
    try {
      // Allow value to be a function so we have same API as useState
      const valueToStore =
        value instanceof Function ? value(storedValue) : value;
      // Save state
      setStoredValue(valueToStore);
      // Save to local storage
      if (typeof window !== "undefined") {
        window.localStorage.setItem(key, JSON.stringify(valueToStore));
      }
    } catch (error) {
      // A more advanced implementation would handle the error case
      console.log(error);
    }
  };
  return [storedValue, setValue];
}
```
