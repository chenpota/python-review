Refer to https://github.com/chenpota/python/tree/master/https-server-client/use-flask-requests

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

import requests


requests.packages.urllib3.disable_warnings(
    requests.packages.urllib3.exceptions.SubjectAltNameWarning
)

try:
    rsp = requests.get(url='https://127.0.0.1:8000', verify='server.pem')
except requests.exceptions.ConnectionError as e:
    print(e)
    exit(1)

print("---url-----------------------------")
print(rsp.url)

print("---status code---------------------")
print(rsp.status_code)

print("---response header-----------------")
print(rsp.headers)

print("---binary response content---------")
print(rsp.content)
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

