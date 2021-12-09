
# SH Weekend Proj 2021-12-11

Alternatively: _The Flaky Router Reboot-and-Alert-inator_

# Design

## Core

 - Python script running inside home; the simplest possible is:

```python
last_ping_success_sec = datetime.now()
while True:
  time.sleep(15)

  try:
    if some_ping('1.1.1.1', timeout_s=5):
      last_ping_success_sec = datetime.now()
  except:
    pass

  if last_ping_success_sec < (datetime.now()-60):
    reboot_router()

```

A more complex variant which would pair with the `Auxiliary Notifications` behaviour below:

```python
last_ping_success_sec = datetime.now()
cloud_server_socket = None

while True:
  time.sleep(15)

  try:
    if not cloud_server_socket:
      cloud_server_socket = connect_to('ws://1.2.3.4/', timeout_s=5)

    if cloud_server_socket:
      cloud_server_socket.send('I am alive!', timeout_s=5)
      last_ping_success_sec = datetime.now() # if .send() blew up we'd be in the except: block and not here
  except:
    pass

  if last_ping_success_sec < (datetime.now()-60):
    reboot_router()

```


## Auxiliary Notifications

 - Python script running on a cloud machine, needs 2 threads:

Thread 1: accept websocket connections and record timestamp of 'I am alive' messages.

```python
last_alive_msg_sec = 0 # global, shared between threads
while True:
  try:
    sock = accept_websocket()
    while True:
      msg = sock.recv()
      if 'I am alive' in msg:
        last_alive_msg_sec = datetime.now()
  except:
    pass

```

Thread 2: poll global state every 15 seconds and send notifications to phones if last 'I am alive' is older than 60s.

```python
while True:
  time.sleep(15)
  now_s = datetime.now()
  if last_alive_msg_sec < (now_s-60):
    send_notifications()
    # Wait at least 5 minutes before yelling again
    last_alive_msg_sec = now_s + (5*60) 
```

# Unanswered Questions / R+D TODOS

 - What is the implementation for some fn `reboot_router()`?
    - Set this up as a controller + use it to send events to devices: https://github.com/jlusiardi/homekit_python
    - Also: https://github.com/homebridge/homebridge/issues/506  /  https://github.com/joelacus/homebridge-tuya-lightbulb-terminal-control

 - Setup websocket server/client: https://pypi.org/project/websockets/
    - This will significantly affect how we write the `^^` example code, as everything will need to be async-aware.






