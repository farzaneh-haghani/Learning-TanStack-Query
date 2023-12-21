# TanStack Query

- We `don't use useEffect for fetch` anymore.
- It helps us mutating data after `POST`,`DELETE` and `PUT`.
- It keeps our data in the `CACHE` and `reuse` it again.
- We don't need to have some state managing such as loading, error, pending ...

### START

1.  install it by `npm install @tanstack/react-query`
2.  wrap our app with `QueryClientProvider`
3.  move the fetch part in separate function
4.  import useQuery hook

5.  config useQuery by passing an object with some parameter such as:

    - `queryKey` which is an array of one or more keys (string, object, nested array ...) to cache the data. So if we try to send same request with same key again, react query be able to reuse the existing data from cache.
    - `queryFn` which should be the fetch function.
    - `staleTime` that controls after which time react query will send such a behind to scene request to get updated data, if it found data in your cache. The default is 0 which means it will used data from the cache but it will then always send such a behind to scene request to get updated data.

      `useQuery({queryKey:['questions'],
      queryFn: fetchQuestions,
      staleTime: 5000

    })`

    it will wait 5000 ms before sending another request.
    if a component was rendered, and a request was sent and within 5 sec the component is rendered again, and the same request would need to be send, React query `would not` send it if it set to 5000.
    But if more than 5 sec the component is rendered again, and the same request would need to be send, React query `is being` send it.

6.  useQuery will get back an object and with destructure it we can pull out
    - `data` which is the response data
    - `isError` which is true if we got back an error response (make sure you throw the error when you check the response in your fetch function)
    - `error` which contain te information about that error such as message
    - `isPending` which is true if the request is still on its way and false if we already got a response
    - `refetch` function which allows us to manually send the same query again.
    - `IsLoading`
7.
