# Mintaro

When you're running a distributed system debugging problems is hard: an error in one service could be caused by problems in completely different services.
In order to debug these problems you want both message transcripts and application logs.
Logging all traffic can be quite expensive, however.

## Mintaro: debugging signal, without the noise

Mintaro is like Sentry for distributed systems: showing you only the information you need to debug a problem.

By configuring Mintaro on your servers you can capture all message traffic and logs that relate to a particular error or an abnormally slow request.
Since only traces that have errors are sent to the Mintaro central server, Mintaro has much lower performance and bandwidth costs compared to systems that store all traffic.

### How it works

Mintaro provides a client SDK which integrates with common web frameworks like Express.js, Sinatra, Django and Spring.
The SDK stores recent HTTP messaging and logs locally, alongside your service.

<img src="ErrorReporting/distributed_logs.png" width="600">

Additionally, the Mintaro SDK allows you to use a trace ID to trace traffic as it flows through your system, so that specific requests and logs can be tied to a particular trace.

When an error occurs or a distributed trace takes too long, the Mintaro SDK notifies a central server that a specific trace had an error, and the central server then notifies all of your services integrated with Mintaro.
Your services then send all HTTP request/response traffic and application log messages for that particular trace to the central server.

When you log in to the Mintaro control UI you will therefore see full message and log traces only for distributed operations that resulted in errors.

<img src="ErrorReporting/error_aggregation.png" width="600">

## Setting up Mintaro

Mintaro works by integrating with your web framework and HTTP client.
For example, if you're using Python with `Flask` you would need to add 3 extra lines of code:

```python
import logging
from flask import Flask

# 1. Mintaro import:
from mintaro import flask_setup, MintaroLogHandler

app = Flask(__name__)

# ... your code here ...

if __name__ == '__main__':
    # 2. Integrate Mintaro with Flask:
    flask_setup(app)
    # 3. Integrate Mintaro with logging
    logging.addHandler(MintaroLogHandler())

    # ... your code here:
    app.listen(8080)
```

In addition, when using `requests` to send HTTP requests you would need to register a custom handler to ensure tracing works.
Instead of calling `requests` directly:

```python
import requests

result = requests.get(url)
```

you should instead use Mintaro's API-compatible wrapper:

```python
from mintaro import requests

result = requests.get(url)
```

If you don't want to use this wrapper you can also just manually set the `X-MINTARO-TRACE` HTTP header to the result of calling `mintaro.get_trace_id()`:

```python
import requests
from mintaro import get_trace_id

result = requests.get(url, {"X-MINTARO-TRACE": get_trace_id()})
```

And that's it!

Similar APIs allow you to configure Mintaro on Ruby (as Rack middleware for Sinatra or Rails) or Javascript (as Express.JS middleware), as well as Django, Java frameworks, etc..

### Interested in being a beta tester?

Sign up [here](https://goo.gl/forms/FLf4iL7kzIU5m11P2).