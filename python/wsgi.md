# WSGI
WSGI stands for Web Server Gateway Interface. The purpose for it is to make sure any python web framework can run in any web server that supports WSGI.

## PEP 0333
### Rationale and Goals
The availability and widespread use of such an API in web servers for Python  would separate choice of framework from choice of web server, freeing users to choose a pairing that suits them, while freeing framework and server developers to focus on their preferred area of specialization.

### Overview
The WSGI interface has two sides: the "server" or "gateway" side, and the "application" or "framework" side. The server side invokes a callable object that is provided by the application side.

the term "a callable" to mean "a function, method, class, or an instance with a \__call__ method". It is up to the server, gateway, or application implementing the callable to choose the appropriate implementation technique for their needs.

### The Application/framework side
```python
def simple_app(environ, start_response):
    """Simplest possible application object"""
    status = '200 OK'
    response_headers = [('Content-type', 'text/plain')]
    start_response(status, response_headers)
    return ['Hello world!\n']


class AppClass:
    """Produce the same output, but using a class

    (Note: 'AppClass' is the "application" here, so calling it
    returns an instance of 'AppClass', which is then the iterable
    return value of the "application callable" as required by
    the spec.

    If we wanted to use *instances* of 'AppClass' as application
    objects instead, we would have to implement a '__call__'
    method, which would be invoked to execute the application,
    and we would need to create an instance for use by the
    server or gateway.
    """

    def __init__(self, environ, start_response):
        self.environ = environ
        self.start = start_response

    def __iter__(self):
        status = '200 OK'
        response_headers = [('Content-type', 'text/plain')]
        self.start(status, response_headers)
        yield "Hello world!\n"
```

### The Server/Gateway Side
The server or gateway invokes the application callable once for each request it receives from an HTTP client, that is directed at the application.

### Middleware: Components that Play Both Sides
A single object may play the role of a server with respect to some application(s), while also acting as an application with respect to some server(s).

### Specification Details
- The application object must accept two positional arguments. For the sake of illustration, we have named them environ and start_response , but they are not required to have these names.
- A server or gateway must invoke the application object using positional (not keyword) arguments.
- The environ parameter is a dictionary object, containing CGI-style environment variables. This object must be a builtin Python dictionary
- The start_response parameter is a callable accepting two required positional arguments, and one optional argument. For the sake of illustration, we have named these arguments status , response_headers , and exc_info , but they are not required to have these names, and the application must invoke the start_response callable using positional arguments
- The status parameter is a status string of the form "999 Message here" , and response_headers is a list of (header_name, header_value) tuples describing the HTTP response header.
- The start_response callable must return a write(body_data) callable that takes one positional parameter: a string to be written as part of the HTTP response body.
- When called by the server, the application object must return an iterable yielding zero or more strings. This can be accomplished in a variety of ways, such as by returning a list of strings, or by the application being a generator function that yields strings, or by the application being a class whose instances are iterable.
- If a call to len(iterable) succeeds, the server must be able to rely on the result being accurate. That is, if the iterable returned by the application provides a working __len__() method, it must return an accurate result.
- If the iterable returned by the application has a close() method, the server or gateway must call that method upon completion of the current request, whether the request was completed normally, or terminated early due to an error.
