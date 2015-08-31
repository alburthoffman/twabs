# Paster
首先python paste是一个WSGI工具包，在WSGI的基础上包装了几层，让应用管理和实现变得方便。

## WSGI
check this link to get more details:
- [wsgi official site](http://wsgi.readthedocs.org/en/latest/)

Basically WSGI expose one interface for Python code to handle request and send back response back.

## example
after install paste package, then you can do this:
```python
if __name__ == "__main__":
    def app(environ, start_response):
        start_response('200 OK', [('content-type', 'text/html')])
        return ['Hello world!']

    from paste import httpserver
    httpserver.serve(app, host="127.0.0.1", port="9092")
```
