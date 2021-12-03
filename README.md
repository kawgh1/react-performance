# React Performance notes

-   ## Dont optimize your code until you've actually measured it's performance otherwise you're shooting in the dark
    -   Not all optimizations improve load speed
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
    -   An **Error Boundary** is a custom fallback **Class** component that is called when an error occurs, Suspense will know to look for this Error Boundary component if it exists and we tell it how to handle accordingly

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
