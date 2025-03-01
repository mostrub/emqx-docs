# Listener Configuration

In EMQX, listener is configured to receive requests from MQTT clients. EMQX supports the following message transfer protocols, including:

- TCP: port  `1883`
- SSL: port `8883`
- Websocket listener: `8083`
- Secure websocket listener: `8084`

:::tip

You can also configure listeners via Dashboard by clicking **Management** -> **Listener** on the left navigation menu of the Dashboard.
However if the listeners are configured from `emqx.conf`, the changes made from the dashboard will only take effect temporarily until EMQX restart.

:::

{% emqxce %}

EMQX has offered more configuration items to serve customized needs better. For details, see [Configuration Manual](https://www.emqx.io/docs/en/v@CE_VERSION@/hocon/).

{% endemqxce %}

{% emqxee %}

EMQX has offered more configuration items to serve customized needs better. For details, see [Configuration Manual](https://docs.emqx.com/en/enterprise/v@EE_VERSION@/hocon/).

{% endemqxee %}

## Configure TCP Listener

TCP listener is a network service that listens for incoming TCP connections on a specific network port. It plays an essential role in establishing and managing connections between clients and EMQX over TCP/IP networks. 

To configure the TCP listener in EMQX, you can add the `listeners.tcp` configuration items in the `emqx.conf` file within the `etc` folder of the EMQX installation directory.

For example, to enable the TCP listener on port `1883`, with a maximum 1,024,000 of concurrent connections allowed by the listener, you can work the code below:

```bash
listeners.tcp.default {
  bind = "0.0.0.0:1883"
  max_connections = 1024000
}
```

where, 

- `listeners.tcp.default` is to enable the listener, and here `default` is the name of the listener, you can change it to your own listener name. 

- `bind` is to set the IP address and port of the listener, here it will listen to all incoming traffic from any IP address on port `1883`. 
- `max_connection` is to set the maximum number of concurrent connections allowed by the listener; default value: `infinity`.

## Configure SSL Listener

SSL listener is a network service that listens for incoming SSL (Secure Sockets Layer) connections. In EMQX, it is used to secure network traffic between a client and EMQX by encrypting the data that is transmitted between them.

To configure the SSL listener in EMQX, you can add the `listeners.ssl` configuration items in the `emqx.conf` file within the `etc` folder of the EMQX installation directory.

For example, to enable the SSL listener on port `8883`, with a maximum 1,024,000 of concurrent connections allowed by the listener:

```bash
listeners.ssl.default {
  bind = "0.0.0.0:8883"
  max_connections = 1024000
  ssl_options {
    cacertfile = "etc/certs/cacert.pem"
    certfile = "etc/certs/cert.pem"
    keyfile = "etc/certs/key.pem"
    verify = verify_none
    fail_if_no_peer_cert = false
  }
}
```

where:

- `listeners.ssl.default` is to enable the listener. 

- `bind` is the IP address and port of the listener, here it will listen to all incoming traffic from any IP address on port `8883`. 
- `max_connection` is the maximum number of concurrent connections allowed by the listener, default value: `infinity`.
- `ssl_options` is the SSL/TLS configuration option for the listener, it has three properties:
  - `cacertfile`: PEM file containing the trusted CA (certificate authority) certificates that the listener uses to verify the authenticity of the client certificates.
  - `certfile`: PEM file containing the SSL/TLS certificate chain for the listener. If the certificate is not directly issued by a root CA, the intermediate CA certificates should be appended after the listener certificate to form a chain.
  - `keyfile`: PEM file containing the private key corresponding to the SSL/TLS certificate.
  - `verify`:  Set 'verify_peer' to verify the authenticity of the clients' certificates, otherwise 'verify_none'.
  - `fail_if_no_peer_cert`: If set to `true`, the server fails if the client does not have a certificate to send, that is, sends an empty certificate. If set to false, it fails only if the client sends an invalid certificate (an empty certificate is considered valid).

## Configure WebSocket Listener

WebSocket listener is a network service that receives and processes messages over WebSocket. WebSocket support in EMQX allows clients to use the WebSocket protocol to connect to EMQX and exchange data in real-time.

To configure the WebSocket listener in EMQX, you can add the `listeners.ws` configuration items in the `emqx.conf` file within the `etc` folder of the EMQX installation directory.

For example, to enable the WebSocket listener on port `8083`, with a maximum 1,024,000 of concurrent connections allowed by the listener:

```bash
listeners.ws.default {
  bind = "0.0.0.0:8083"
  max_connections = 1024000
  websocket.mqtt_path = "/mqtt"
}
```

where:

- `listeners.ws.default` is to enable the listener. 

- `bind` is the IP address and port of the listener, here it will listen to all incoming traffic from any IP address on port `8083`. 
- `max_connection` is the maximum number of concurrent connections allowed by the listener, default value: `infinity`.
- `websocket.mqtt_path` is to set the path to the WebSocket’s MQTT protocol, which is `/mqtt` by default. 

## Configure Secure WebSocket Listener

A secure WebSocket listener is a WebSocket listener that uses the Secure Sockets Layer (SSL) or Transport Layer Security (TLS) protocol to encrypt the data exchanged between a WebSocket client and the broker. In EMQX, the secure WebSocket listener is an important security measure to protect sensitive data exchanged between WebSocket clients and EMQX> 

To configure the secure WebSocket listener in EMQX, you can add the `listeners.wss` configuration items in the `emqx.conf` file within the `etc` folder of the EMQX installation directory.

For example, to enable the Secure WebSocket listener on port `8084`, with a maximum 1,024,000 of concurrent connections allowed by the listener:

```bash
listeners.wss.default {
  bind = "0.0.0.0:8084"
  max_connections = 1024000
  websocket.mqtt_path = "/mqtt"
  ssl_options {
    cacertfile = "etc/certs/cacert.pem"
    certfile = "etc/certs/cert.pem"
    keyfile = "etc/certs/key.pem"
  }
}
```

where:

- `listeners.wss.default` is to enable the listener. 

- `bind` is the IP address and port of the listener, here it will listen to all incoming traffic from any IP address on port `8084`. 
- `max_connection` is the maximum number of concurrent connections allowed by the listener, default value: `infinity`.
- `websocket.mqtt_path` is to set the path to the WebSocket’s MQTT protocol, which is `/mqtt` by default. 
- `ssl_options` is the SSL/TLS configuration option for the listener, it has three properties:
  - `cacertfile`: PEM file containing the trusted CA (certificate authority) certificates that the listener uses to verify the authenticity of the client certificates.
  - `certfile`: PEM file containing the SSL/TLS certificate chain for the listener. If the certificate is not directly issued by a root CA, the intermediate CA certificates should be appended after the listener certificate to form a chain.
  - `keyfile`: PEM file containing the private key corresponding to the SSL/TLS certificate.

<!--To add QUIC-->

<!--To add code sample for adding multiple listeners.-->

## Link Listener to a Configuration zone

Each listener in EMQX is associated with a zone, which by default is set to a logical zone named `default`.

When a listener is linked to a specific zone, MQTT clients connected to that listener inherit the settings from that zone.

For more information, see the [Zone Override](./configuration.md#zone-override) section in the configuration documentation.

