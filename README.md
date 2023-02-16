# Post Message POC

This project is a POC for using [PostMessageAPI](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) in iframes.

The goal is to safely communicate between windows (the one hosting the iframe and the one within the iframe).

## Install and Serve

To serve the examples and see the POC results:

> npm install
> npm run serve:all


This will serve three example apps:

| App | URL                   |
|-----|-----------------------|
| A   | http://localhost:8001 |
| B   | http://localhost:8002 |
| U   | http://localhost:8003 |

## How it works

In `src` folder there is 3 simple example apps, that will implement and help understand better how to send messages to another window and how to filter received messages. 

The apps are: `A`, `B` and `U`. Although all three apps can be served alone, for this POC propose they must be served togheter, because:

- `A` has an iframe to embed `B` in its body
- `B` is embed in `A` and recoginizes it as a safe source to receive messages
- `U` also has an iframe to embed `B` in its body, but it simulates an **unkown/unsafe soruce** 

`A` and `B` recoginizes one another as safe sources and will communicate using `PostMessage API`. Meanwhile, `U` is not recognized by them, and when it tries to communicate with `B`, the messages will not be consider safe and will be ignored. 

In addition to that, everytime a message is sent or received by a window, it is added in its "Messages Log" list.

## Results

### Proves

With this project, I was able to prove:

:incoming_envelope: An app with an iframe in its body can communicate with the iframe's `contentWindow`
:incoming_envelope: An app being hosted in an iframe can communicate to it's `parent.window`
:shield: An app receiving messages with `PostMessage API` can filter messages using the `origin` attribute, making it possible to secure itself from unknown/unspected sources.

### Understanding the API

When checking the API docs, I found it really confusing to understand the [examples](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage#examples), that's why I decided to write this POC. So, I did this project to understand the API better and test some features. Here is a sample of how I used the API:

### Sending Messages

Considering `target` as the windows **receiving** the message, we can afirm that `postMessage` method has the pattern: 

> `targetWindowHtmlElement.postMessage(data, targetWindowOrigin)`

As we are using iframes, the `targetWindowHtmlElement` would be `iframe.contentWindow` and/or `iframe.parent`. So, considering that `A` has an iframe embeding `B`:


#### Sending messages from `A` to `B`

```js
// in A app
let windowBIframe = document.getElementById("window-b-iframe")
let windowB;

windowBIframe.onload = () => {
  windowB = windowBIframe.contentWindow;
}
      
windowB.postMessage(content, WINDOW_B_URL);
```

#### Sending messages from `B` to `A`

```js
// in B app
const windowB = window;
const windowA = windowB.parent;

windowA.postMessage(content, WINDOW_A_URL);
```


#### `A` receiving messages from `B` 

```js
// in A app
windowA.addEventListener('message', (event) => {
  // checking for safe sources
  if (event.origin === WINDOW_B_URL) {
    console.log(event)     
  }
})
```

#### `B` receiving messages from `A`

```js
// in B app
windowB.addEventListener('message', (event) => {
  // checking for safe sources
  if (event.origin === WINDOW_A_URL) {
    console.log(event)     
  }
})
```
