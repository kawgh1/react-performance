# React Performance notes

-   These notes deal primarily with this repo, but are general for all react applications
-   https://github.com/kawgh1/react-crown-clothing-optimized

-   ## Dont optimize your code until you've actually measured it's performance otherwise you're shooting in the dark
    -   Not all optimizations improve load speed
    -   Some optimizations can actually make loading slower and/or add more code complexity than an insignificant improvement is worth
-   ## Code Splitting and Chunking / Bundling

    -   When we load the homepage, we don't want to load every page, every resource. We just want to load what the homepage requires to display.
    -   We want each page to load only what is necessary for that page.
    -   https://create-react-app.dev/docs/code-splitting/
    -   It's best to chunk at the Page level, lazy loading individual components generally does not yield much performance benefit and may make the code more complex than is needed - unless there is a very large heavy performing component on a page that is not run until a user action (click on something runs a large component) - then maybe, otherwise no
    -   Even chunking a very large component on a page, you would want to test first - is loading this component actually slowing down the page? If not, no reason to chunk (lazy load) it

-   ## Dynamic importing

    -   User react lazy, which is just imported from react
        -   import React, { useEffect, lazy } from 'react';
    -   We wrap our Routes with Lazy + Suspense and what this means is that when a user loads the Homepage Route, they only get the Homepage chunk, when they load the Shop Route they only get the Shop chunk
    -   Thus we are downloading CHUNKS at the ROUTE level instead of the TOP level (ie. one mega mainchunk all at once on the homepage dump)

-   ## Error Handling and Error Boundaries

    -   Using React Suspense and Lazy Loading means we need to pay extra attention to error handling. Suspense and Lazy Loading don't know how to handle errors gracefully and can result in a page or component getting stuck in a perpetual "Loading Screen" with no resolution and no indication to the user an error has occurred.
    -   An **Error Boundary** is a custom fallback **Class** component that is called when an error occurs inside any page nest inside Suspense tags, Suspense will know to look for this Error Boundary component if it exists and we tell it how to handle accordingly
    -   https://reactjs.org/docs/error-boundaries.html
    -   https://www.kapwing.com/404-illustrations

            <Header />
                <Switch>
                    <ErrorBoundary>
                        <Suspense fallback={<Spinner />}>
                            <Route exact path="/" component={HomePage} />
                            <Route path="/shop" component={ShopPage} />
                            <Route
                                exact
                                path="/checkout"
                                component={CheckoutPage}
                            />
                            <Route
                                exact
                                path="/signin"

                            />
                        </Suspense>
                    </ErrorBoundary>
                </Switch>

    ![error-boundary](https://raw.githubusercontent.com/kawgh1/react-performance/master/error-boundary-example.png)

-   ## React.memo() and PureComponents

    -   consider React.memo() on functional components to prevent unnecessary re-rendering using memoization
    -   https://reactjs.org/docs/react-api.html#reactmemo
    -   https://reactjs.org/docs/react-api.html#reactpurecomponent

    -   ## NOTE
        -   **If you're using a Class Component use React PureComponent. If you're using a Functional Component, use React.memo()**

-   ## Using React Dev Tools on Chrome

    -   Use Profiler tab to record a click through of a page or sequence and it will show you the load time for each component rendered, how many re-renders, etc.
    -   https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi/related?hl=en

-   ## React Performance with Redux

    -   Since we are using Redux and Reselect in our application, alot of our components get their props not from their parent components, but through Redux and the Connect Higher Order Component (HOC) Pattern we are using.
    -   Because a lot of our prop derivation is decided by `MapStateToProps` and Reselect `createSelector` and this means a lot of our props are already memoized \*\*meaning that unless the actual props coming into our Selectors have changed, we will not actually get new props, meaning our components will not rerender.\*\* This is an out of the box improvement over native React because we are using the right Redux libraries to optimize our code
    -   Since this is the case, just by using `MapStateToProps` and Reselect, a lot of optimization has already been done for us and we should only look at optimizing code further where an actual slow down is occurring.
    -   One example of a place we can optimize is when adding items to cart, our cart component is re-rendering **every** cart item every time a new cart item is added. This was discovered used the React Dev Tools Profiler and adding items to the cart, reviewing the output.
        -   We fix this by simply memoizing the Cart Item Component like so:
            -   `export default React.memo(CartItem);`
    -   Can also memoize individual functions using the `Callback()` hook

            const logName = useCallback(() =>
                console.log('this function only renders once and is simply recalled
                from memory on any later broader state re-renders'),
            [])

        -   `Callback()` takes the function to be memoized as its first parameter and an array of any dependencies as the second
            -   It will only re-render if the dependent variables of the function change from state to state, rather than on every state change

    -   `useMemo()` is a similar memoization hook

        -   We use `Callback()` when we want a memoized function and we use `useMemo()` when we want a memoized value - for example, a value that is computationally expensive to generate (heavy load on the browser) - rather than re-computing the value on each re-render, `useMemo()` will simply recall the memory cached value of the last computation render, if no dependency variables have changed

                const doSomethingComplicated = useMemo(() => {
                    console.log('something complex');
                    return((count1 - 1000) % 12.4) - 72123 -4827
                }, [count1]);
