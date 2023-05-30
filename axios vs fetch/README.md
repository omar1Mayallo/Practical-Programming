# Axios vs Fetch

One of the fundamental tasks of any web application is to communicate with servers through the HTTP protocol. This can be easily achieved using Fetch or Axios. Fetch and Axios are very similar in functionality. Some developers prefer Axios over built-in APIs for its ease of use.

In this article, we implement the main differences between `axios` and `fetch`, and in the end, we give a conclusion of these differences.

## What is it
- [**_axios_**](https://axios-http.com/docs/intro)
> Axios is a `promise-based` HTTP Client for node.js and the browser. It is isomorphic (= it can run in the browser and nodejs with the same codebase). On the server-side it uses the native node.js `http` module, while on the client (browser) it uses [XMLHttpRequests](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest).

- [**_fetch_**](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) 
> The Fetch API provides an interface for fetching resources (including across the network). It is a more powerful and flexible replacement for [XMLHttpRequests](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest).

## Backward compatibility
  - **_axios_**
    - Axios uses the XMLHttpRequest (XHR) object internally to make HTTP requests. This means that Axios has built-in support for older browsers that do not have the Fetch API available.<br/><br/>
    - Axios support `node.js`

  - **_fetch_**
    - Fetch on the other hand, only supports Chrome 42+, Firefox 39+, Edge 14+, and Safari 10.3+ (you can see the full compatibly table on [CanIUse.com](https://caniuse.com/fetch)).<br/><br/>
    - The Fetch API comes bundled with NodeJS v17.5 and later, Before that we need to use packages like `node-fetch` to use it in `node.js` environment.<br/><br/>
    - If your only reason for using Axios is backward compatibility, you don’t really need an HTTP library. Instead, you can use `fetch()` with a polyfill like [this](https://github.com/github/fetch) to implement similar functionality on web browsers that do not support `fetch()`.
    
## Automatic JSON data handling in response
  - **_axios_**
    - Axios automatically parses the response data based on the specified `Content-Type` header and provides it as the data property of the response object. It supports parsing `JSON`, `XML`, and other formats out of the box.<br/><br/>
    
    ``` js
    axios.get('https://api.example.com/data')
    .then(response => {
      console.log(response.data);
    }, error => {
      console.log(error);
    });
    ```
  - **_fetch_**
    - With Fetch, you need to manually parse the response using the appropriate method (`json()`, `text()`, `blob()`, etc.) based on the expected response format.<br/><br/>
    
    ``` js
    fetch('https://api.example.com/data')
    .then(response => response.json()) // one extra step
    .then(data => {
      console.log(data);
    })
    .catch(error => {
      console.error(error);
    });
    ```
    
## Request and Response Interceptors
- **_axios_**
  - One of the key features of Axios is its ability to intercept HTTP requests. HTTP interceptors come in handy when you need to examine or change HTTP requests from your application to   the server or vice versa (e.g., logging, authentication, or retrying a failed HTTP request).<br/> With interceptors, you won’t have to write separate code for each HTTP request. HTTP interceptors are helpful when you want to set a global strategy for how you handle request and response.<br/><br/>
  - In this example, Axios allows you to define request and response interceptors using `axios.interceptors.request.use()` and `axios.interceptors.response.use()`, respectively. The request interceptor allows you to modify the request configuration before it is sent, while the response interceptor allows you to modify the response data after it is received.<br/><br/>
  
  ``` js
  import axios from 'axios';

  // Request interceptor
  axios.interceptors.request.use(
    config => {
      console.log('Request interceptor - Modify request config:', config);
      // You can modify the request config here (e.g., add headers)
      return config;
    },
    error => {
      console.error('Request interceptor - Error:', error);
      return Promise.reject(error);
    }
  );

  // Response interceptor
  axios.interceptors.response.use(
    response => {
      console.log('Response interceptor - Modify response data:', response.data);
      // You can modify the response data here (e.g., transform, filter)
      return response;
    },
    error => {
      console.error('Response interceptor - Error:', error);
      return Promise.reject(error);
    }
  );

  // Make a request
  axios.get('https://api.example.com/data')
    .then(response => {
      console.log('Response:', response.data);
    })
    .catch(error => {
      console.error('Error:', error);
    });
  ```
  
- **_fetch_**
  - Fetch does not provide built-in interceptors. If you need to modify requests or responses, you would need to manually modify the request or response objects before using them.<br/><br/>
  - In the example above, we modify the request by creating a new [`Request`](https://developer.mozilla.org/en-US/docs/Web/API/Request) object with custom headers and log the modified request. After receiving the response, we can then modify the response data within the `.then()` block.<br/><br/>
  
   ``` js
   const url = 'https://api.example.com/data';

  // Fetch with request interceptor
  const modifiedRequest = new Request(url, {
    method: 'GET',
    headers: {
      'Content-Type': 'application/json',
    },
  });

  console.log('Modified Request:', modifiedRequest);

  // Fetch request
  fetch(modifiedRequest)
    .then(response => {
      // Response interceptor
      console.log('Response:', response);
      return response.json();
    })
    .then(data => {
      console.log('Response Data:', data);
      // You can modify the response data here (e.g., transform, filter)
    })
    .catch(error => {
      console.error('Error:', error);
    });
   ```
## Concurrency(Simultaneous requests)
Concurrency refers to the ability to handle multiple requests simultaneously.

- **_axios_**
  - In Axios, you can use the `axios.all()` method to make multiple requests concurrently. The method takes an array of requests and returns a single promise that resolves when all requests have been completed. then use `axios.spread()` to assign the properties of the response array to separate variables : <br/><br/>
  
  ``` js
  import axios from 'axios';

  // Make multiple requests concurrently
  axios.all([
    axios.get('https://api.example.com/data1'),
    axios.get('https://api.example.com/data2'),
    axios.get('https://api.example.com/data3')
  ])
    .then(axios.spread((response1, response2, response3) => {
      console.log('Response 1:', response1.data);
      console.log('Response 2:', response2.data);
      console.log('Response 3:', response3.data);
    }))
    .catch(error => {
      console.error('Error:', error);
    });
  ```
  
- **_fetch_**
  - In Fetch, you can achieve concurrency by creating an array of fetch promises and using `Promise.all()` to execute them concurrently. The `Promise.all()` method takes an array of promises and returns a new promise that resolves when all promises in the array have been resolved. then handle the response by using an async function : <br/><br/>
  
  ``` js
  const urls = [
  'https://api.example.com/data1',
  'https://api.example.com/data2',
  'https://api.example.com/data3'
  ];

  // Create an array of fetch promises
  const fetchPromises = urls.map(url => fetch(url));

  // Execute promises concurrently using Promise.all()
  Promise.all(fetchPromises)
    .then(responses => {
      // Extract response data from each response object
      const responseDataPromises = responses.map(response => response.json());
      return Promise.all(responseDataPromises);
    })
    .then(responseData => {
      console.log('Response 1:', responseData[0]);
      console.log('Response 2:', responseData[1]);
      console.log('Response 3:', responseData[2]);
    })
    .catch(error => {
      console.error('Error:', error);
    });
  ```
  
<hr/>

### Conclusion
Overall, Axios provides a more comprehensive and user-friendly API with advanced features like automatic error handling, interceptors, and more. It simplifies common tasks and provides a consistent experience across different environments. Fetch, on the other hand, is a native browser function and has a simpler syntax, but it may require additional handling for error management and cross-origin requests.

When choosing between Axios and Fetch, consider your project's requirements, the need for browser support, and the level of convenience and features you expect from the library. However, if you prefer to stick with native APIs, nothing stops you from implementing Axios features.

| Aspect | Axios | Fetch |
| :---        |    :----:   |    :----:   |
| API and Syntax | Third-party library with a consistent and intuitive API | Native JavaScript function with a simpler syntax |
| Error Handling | Built-in error handling for non-2xx status codes, Transforms certain types of errors into descriptive objects | Network errors rejected; non-2xx codes not treated as errors |
| Browser Support | Supports a wide range of browsers, including older versions | Supported in most modern browsers, lacks support in some older versions |
| Cancellation | Built-in cancellation support with cancel tokens(deprecated) now it uses `AbortController` | No built-in cancellation support, but can be achieved using `AbortController` |
| Response Formats | Automatic parsing of response based on `Content-Type` | Manual parsing of response using appropriate method |
| Customization and Configuration | Extensive configuration options	 | Fewer built-in configuration options |
| Concurrency | Supports concurrent requests | No built-in support for concurrent requests |
| Library Size | Standalone library, adds to bundle size | Native browser API, no additional library required |
| Interceptors | Provides interceptors for request/response manipulation	 | No built-in interceptors, requires manual request/response manipulation |

<hr/>

#### resources
- [https://axios-http.com/docs/intro](https://axios-http.com/docs/intro)
- [https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [https://stackoverflow.com/questions/50277504/is-there-any-reasons-to-use-axios-instead-es6-fetch](https://stackoverflow.com/questions/50277504/is-there-any-reasons-to-use-axios-instead-es6-fetch)
- [https://blog.logrocket.com/axios-vs-fetch-best-http-requests/#backward-compatibility](https://blog.logrocket.com/axios-vs-fetch-best-http-requests/#backward-compatibility)
- [https://blog.logrocket.com/how-to-make-http-requests-like-a-pro-with-axios/#new_tab](https://blog.logrocket.com/how-to-make-http-requests-like-a-pro-with-axios/#new_tab)
- [https://www.geeksforgeeks.org/difference-between-fetch-and-axios-js-for-making-http-requests/](https://www.geeksforgeeks.org/difference-between-fetch-and-axios-js-for-making-http-requests/)
- [https://www.scaler.com/topics/nodejs/node-js-fetch/](https://www.scaler.com/topics/nodejs/node-js-fetch/)

