# TanStack Query

- We `don't use useEffect for fetch` anymore.
- It helps us mutating data after `POST`,`DELETE` and `PUT`.
- It keeps our data in the `CACHE` and `reuse` it again.
- We don't need to have some state managing such as loading, error, pending ...

## START

1.  Install it by `npm install @tanstack/react-query`
2.  Wrap our app with `QueryClientProvider`

## Get ALL

1.  Move the fetch part in separate function
2.  Import useQuery hook

3.  Config useQuery by passing an object with some parameter such as:

    - `queryKey` which is an array of one or more keys (string, object, nested array ...) to cache the data. So if we try to send same request with same key again, react query be able to reuse the existing data from cache.

    - `queryFn` which should be the fetch function.

    - `staleTime` that controls after which time react query will send such a behind to scene request to get updated data, if it found data in your cache. The default is 0 which means it will used data from the cache but it will then always send such a behind to scene request to get updated data.

      ```jsx
      useQuery({
        queryKey: ["questions"],
        queryFn: fetchQuestions,
        staleTime: 5000,
      });
      ```

      it will wait 5000 ms before sending another request.
      if a component was rendered, and a request was sent and within 5 sec the component is rendered again, and the same request would need to be send, React query`would not`send it if it set to 5000.
      But if more than 5 sec the component is rendered again, and the same request would need to be send, React query`is being` send it.

    - `gcTime` the garbage collection time which controls how long the data in the cache will be kept around and after it will be discarded so if the component needs to render again, there would be no cached data and react would send a new request to get data before it can show anything. The default are 5 minutes and we could reduce it if we want.

4.  useQuery will get back an object and with destructure it we can pull out
    - `data` which is the response data
    - `isError` which is true if we got back an error response (make sure you throw the error when you check the response in your fetch function)
    - `error` which contain te information about that error such as message
    - `refetch` function which allows us to manually send the same query again.
    - `isPending` which is true if the request is still on its way and false if we already got a response
    - `IsLoading` same as isPending but the only difference is isLoading will not be true if the query is just disabled.

## Get One

1.  We can reuse the same fetch function for get one as well as get all, but we need to construct dynamic url.
2.  We should change queryFn to callback function and send the searchTerm as a parameter.

    ```jsx
    // part of fetch function
    export async fetchQuestions(searchTerm)=>{
      let url ="http://localhost:4000/questions"
      if(searchTerm){
        url +=`?search=${searchTerm}`
      }
      const res=await fetch(url)
    }

    // component
    useQuery({
    queryKey: ['questions',{ search: searchTerm }],
    queryFn:()=>fetchQuestions(searchTerm),
    })
    ```

3.  If we use the same queryKey as get all, react query will fetch all data that we don't need all, our goal is `fetching only questions` that match with our search term. Now In the queryKey we should use extra object to make it clear which kind of other value that we have.

4.  But it is not working because if we don't have searchTerm, we still have `?search=[object object]` in the request url. why?
    Because always useQuery hook passes some default data to all query function which is an object with some keys `{meta,queryKey,signal,get signal}`

    - `What is signal?` Signal is required for aborting the request for example if the user navigate from that page to another page.

5.  Therefore we should accept that object in the fetch function and destructure it to pull out some of those things like `signal`.
    It's good to have signal because we can make sure that the request is being send will abort if react query thinks that it should be aborted.
6.  We can pass signal as an configuration object to the `await fetch` function by adding as a second argument. So the browser can use abort signal internally to stop the request if it receives that.

7.  Now we need to access searchTerm correctly in the fetch function so we need to make sure that we pass searchTerm as an object in the queryFn. Because we used callback function so we need to get signal in the queryFn first and then pass it with searchTerm

```jsx
export async fetchQuestions({ signal,searchTerm })=>{
  let url ="http://localhost:4000/questions"
  if(searchTerm){
    url +=`?search=${searchTerm}`
  }
  const res=await fetch(url,{ signal:signal })
}

// component
useQuery({
queryKey: ['questions',{ search: searchTerm }],
queryFn:({ signal })=>fetchQuestions({ signal,searchTerm }),
})
```

Now we have another problem when the user visiting the app for first time, we will see all data twice, because we fetch them one time to show all data and another time for searching component and because there is no searchTerm, it will fetch all data again.

- We need to make sure that we don't send the query and `disable` it if we did not enter any search at all. We can add `enable` property in configuration object. If it was `false`, the query will be disabled and by default is true. We could set `searchTerm !== ""`

```jsx
useQuery({
  queryKey: ["questions", { search: searchTerm }],
  queryFn: ({ signal }) => fetchQuestions({ signal, searchTerm }),
  enable: searchTerm !== "",
});
```

Now we have another problem that we will see loading snipper for the first visiting. We don't want to see any data at the first. We want to see the relevant data after the user searched and then if the user delete the searchTerm, it shows all data.
So we can't check it by empty string. We can set the initial value of useState for searchTerm be `undefined`

`const [searchTerm, setSearchTerm]=useState();`

```jsx
useQuery({
  queryKey: ["questions", { search: searchTerm }],
  queryFn: ({ signal }) => fetchQuestions({ signal, searchTerm }),
  enable: searchTerm !== undefined,
});
```

Now we don't have any data for the first time but reactQuery treats it as pending, and we still see loading snipper.
Now we can pull out isLoading instead of isPending.
