# MQTT

The embedded [MQTT Broker](/features/mqttBrokerProxy.md) provides a standard MQTT broker interface that can be accessed by clients written in any language.

## Interactive Producer and Consumer

Easiest way to interact with the MQTT Broker is via the [MQTTX](https://mqttx.app/) app.

## Sample Producer in Python 3

Before running the code make sure you install the libraries:

```bash
pip install paho-mqtt
```

The below script will attempt to connect to your broker, and if successful, it will publish the message and then immediately disconnect:

```python
import paho.mqtt.client as mqtt
import time
import sys

# --- Configuration ---
BROKER_ADDRESS = "localhost"
BROKER_PORT = 1883
TOPIC = "test/topic"
MESSAGE = "Hello, MQTT from Python!"
# ---------------------

def on_connect(client, userdata, flags, rc):
    """
    The callback for when the client receives a CONNACK response from the server.
    rc (return code) 0 means success.
    """
    if rc == 0:
        print("Connected successfully to MQTT broker.")
        # Once connected, publish the message
        publish_message(client)
    else:
        print(f"Connection failed with code {rc}. Check if the broker is running.")
        sys.exit(1)

def on_publish(client, userdata, mid):
    """
    The callback for when a message has been published successfully.
    mid is the message ID of the published message.
    """
    print(f"Message published (mid: {mid}).")
    # Disconnect after publishing the message
    client.disconnect()
    # A small sleep to let the disconnect complete before the script ends
    time.sleep(0.5)

def publish_message(client):
    """Publishes the configured message to the configured topic."""
    print(f"Attempting to publish '{MESSAGE}' to topic '{TOPIC}'...")
    # The publish call returns a result object. We use QoS 1.
    # QoS 1 means the message is delivered at least once.
    client.publish(TOPIC, MESSAGE, qos=1)

def main():
    """Main function to set up and run the MQTT client."""
    print(f"Starting MQTT client. Target: {BROKER_ADDRESS}:{BROKER_PORT}")

    # Create a new MQTT client instance (using API version 1 for broader compatibility)
    client = mqtt.Client(mqtt.CallbackAPIVersion.VERSION1)

    # Assign callback functions
    client.on_connect = on_connect
    client.on_publish = on_publish

    try:
        # Connect to the broker. The last argument is the keepalive time in seconds.
        client.connect(BROKER_ADDRESS, BROKER_PORT, 60)

        # Start the network loop in a non-blocking thread.
        # This is necessary for callbacks (like on_connect) to fire.
        client.loop_start()

        # Keep the main thread alive briefly to allow the connection, publication,
        # and disconnection process to complete via the loop_start thread.
        print("Waiting for connection and publish to complete...")
        while client.is_connected():
            # Check if the loop is still running, exit after a timeout or successful disconnect
            time.sleep(0.1)

    except ConnectionRefusedError:
        print("FATAL ERROR: Connection refused. Is your MQTT broker running on localhost:1873?")
    except Exception as e:
        print(f"An unexpected error occurred: {e}")
    finally:
        # Ensure the loop stops if it was started
        client.loop_stop()
        print("Client loop stopped. Script exit.")

if __name__ == "__main__":
    main()

```

## Sample Listener in Python 3

The following script connects, subscribes, and waits indefinitely for messages.

```python
import paho.mqtt.client as mqtt
import sys
import time

# --- Configuration ---
BROKER_ADDRESS = "localhost"
BROKER_PORT = 1873
TOPIC = "test/topic"
# ---------------------

def on_connect(client, userdata, flags, rc):
    """
    The callback for when the client receives a CONNACK response from the server.
    rc (return code) 0 means success.
    """
    if rc == 0:
        print("Connected successfully to MQTT broker.")
        # Once connected, subscribe to the topic
        client.subscribe(TOPIC)
        print(f"Subscribed to topic: '{TOPIC}'. Waiting for messages...")
    else:
        print(f"Connection failed with code {rc}. Check if the broker is running.")
        sys.exit(1)

def on_message(client, userdata, msg):
    """
    The callback for when a PUBLISH message is received from the server.
    This function processes the incoming message payload.
    """
    # Decode the payload from bytes to a string
    payload_str = msg.payload.decode()
    print("-" * 30)
    print(f"[{time.strftime('%H:%M:%S')}] Message Received!")
    print(f"Topic: {msg.topic}")
    print(f"Payload: {payload_str}")
    print(f"QoS: {msg.qos}")
    print("-" * 30)


def main():
    """Main function to set up and run the MQTT subscriber client."""
    print(f"Starting MQTT subscriber client. Target: {BROKER_ADDRESS}:{BROKER_PORT}")

    # Create a new MQTT client instance (using API version 1)
    client = mqtt.Client(mqtt.CallbackAPIVersion.VERSION1)

    # Assign callback functions
    client.on_connect = on_connect
    client.on_message = on_message

    try:
        # Connect to the broker
        client.connect(BROKER_ADDRESS, BROKER_PORT, 60)

        # Blocking call that processes network traffic, dispatches callbacks,
        # and handles reconnecting. It blocks the main thread.
        print("Client running. Press Ctrl+C to stop.")
        client.loop_forever()

    except ConnectionRefusedError:
        print("FATAL ERROR: Connection refused. Is your MQTT broker running on localhost:1873?")
    except KeyboardInterrupt:
        print("\nSubscriber stopped by user.")
    except Exception as e:
        print(f"An unexpected error occurred: {e}")
    finally:
        client.disconnect()
        print("Client disconnected. Script exit.")

if __name__ == "__main__":
    main()

```