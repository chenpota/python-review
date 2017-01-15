Refer to https://github.com/chenpota/python/tree/master/https-server-client/use-flask-urllib

# Create private key and certificate

    $ openssl req -new -x509 -keyout server.key -out server.pem -days 365 -nodes -subj '/C=TW/ST=Taiwan/L=Taipei/CN=127.0.0.1/emailAddress=test@test.com'

# server.py

```python
#!/usr/bin/env python3

import flask


app = flask.Flask(__name__)


@app.route('/', methods=['GET'])
def my_get_handler():
    return flask.make_response(
        flask.jsonify(
            {
                'id': 'XXX-XXXX',
                'content': 'TBD'
            }
        ),
        200
    )

if __name__ == '__main__':
    app.run(
        host='127.0.0.1',
        port=8000,
        ssl_context=('server.pem', 'server.key')
    )
```

# client.py

```python
#!/usr/bin/env python3

import json
import ssl
import urllib.error
import urllib.parse
import urllib.request


req = urllib.request.Request("https://127.0.0.1:8000/")

ctx = ssl.create_default_context(cafile='server.pem')
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_REQUIRED

try:
    rsp = urllib.request.urlopen(req, context=ctx)
except urllib.error.HTTPError as e:
    print(e)
    exit(1)

print("---status code---------------------")
print(rsp.getcode())

print("---response header-----------------")
print(rsp.info())

print("---binary response content---------")
print(rsp.read())

rsp.close()
```

# Server side operation
    root@ce3454be9c1f:/https-test# openssl req -new -x509 -keyout server.key -out server.pem -days 365 -nodes -subj '/C=TW/ST=Taiwan/L=Taipei/CN=127.0.0.1/emailAddress=test@test.com'
    Generating a 2048 bit RSA private key
    ..........+++
    ...........................................................................+++
    writing new private key to 'server.key'
    -----
    root@ce3454be9c1f:/https-test#
    root@ce3454be9c1f:/https-test# ./server.py 
     * Running on https://127.0.0.1:8000/ (Press CTRL+C to quit)
    127.0.0.1 - - [15/Jan/2017 00:22:38] "GET / HTTP/1.1" 200 -
    
# Client side operation
    root@ce3454be9c1f:/https-test# ./client.py 
    ---status code---------------------
    200
    ---response header-----------------
    Content-Type: application/json
    Content-Length: 44
    Server: Werkzeug/0.11.15 Python/3.5.2
    Date: Sun, 15 Jan 2017 00:22:38 GMT
    
    
    ---binary response content---------
    b'{\n  "content": "TBD", \n  "id": "XXX-XXXX"\n}\n'
    root@ce3454be9c1f:/https-test# 

