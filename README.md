# Instant Flask Server Deployment
Quickly deploy an API endpoint with VM, Certbot, Gunicorn, and Flask

## Dependencies
1. Create a virtual machine with HTTP, HTTPs exposed (Ubuntu 20)
2. Run the following
```sudo apt update && sudo apt-get install python3 && sudo snap install --classic certbot && sudo ln -s /snap/bin/certbot /usr/bin/certbot && sudo apt-get install python3-pip && pip install flask && pip install flask-cors && pip install gunicorn```
3. Create a new Python file called ```server.py``` with the following code
````
from flask import Flask, request, jsonify
from flask_cors import CORS
import logging
from http.server import BaseHTTPRequestHandler

app = Flask(__name__)
CORS(app)

@app.route('/')
def index():
    gunicorn_error_logger = logging.getLogger('gunicorn.error')
    app.logger.handlers.extend(gunicorn_error_logger.handlers)
    app.logger.setLevel(logging.DEBUG)
    app.logger.debug('Logging established')
    return 'Hello, World!'

if __name__ == '__main__':
    app.run(host='0.0.0.0')
````

# Connecting Domain
Go to your DNS provider and set up an A record with @ and the VM IP address

# Verifying HTTPS
1. Run ```sudo certbot certonly --webroot```
2. Give read access (there might be a more efficient way to do this)
````
sudo chmod 755 /etc &&
sudo chmod 755 /etc/letsencrypt &&
sudo chmod 755 /etc/letsencrypt/live &&
sudo chmod 755 /etc/letsencrypt/live/example.com &&
sudo chmod 755 /etc/letsencrypt/live/example.com/privkey.pem &&
sudo chmod 755 /etc/letsencrypt/live/example.com/fullchain.pem &&
sudo chmod 755 /etc/letsencrypt/archive &&
sudo chmod 755 /etc/letsencrypt/archive/example.com &&
sudo chmod 755 /etc/letsencrypt/archive/example.com/fullchain1.pem &&
sudo chmod 755 /etc/letsencrypt/archive/example.com/privkey1.pem 
````

# Running the Server
1. Note: this doesn't use WSGI
2. Run ```gunicorn --certfile=/etc/letsencrypt/live/example.com/fullchain.pem --keyfile=/etc/letsencrypt/live/example.com/privkey.pem --bind 0.0.0.0:8000 server:app```
3. You can stop the server with ```pkill gunicorn```

You can now GET and POST with endpoint https://example.com:8000
