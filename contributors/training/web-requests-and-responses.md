
# Web Requests and Responses

## Overview

In Python, we obtain information from sources on the internet through web requests and responses. We send a request to a URL/IP address, and receive a response from the server.

### Types of requests

Reference: [HTTP request methods [MDN]](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)

Although there are many types of requests that we can send, `GET` and `POST` are the two most commonly used.

### Types of responses

Reference: [HTTP response status codes [MDN]](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)

A response from the server includes a status code indicating the success or failure of a response. This should be checked in the code and appropriately handled.

## Sending requests in Python

### Using the Python Requests Package

The [`requests`](https://requests.readthedocs.io/en/latest/) library in Python makes it easy to send HTTP requests. See the [Quickstart page](https://requests.readthedocs.io/en/latest/user/quickstart/) for some usage examples.

## Handling response data

Many web APIs use [JSON](csv-and-json.md) as a data exchange format, which Python ahs native support for.

For websites that do not have a web API, the response data is most likely in HTML format, which can be parsed with libraries like [html.parser](https://docs.python.org/3/library/html.parser.html) (built-in) or [beautifulsoup](https://beautiful-soup-4.readthedocs.io/en/latest/).
