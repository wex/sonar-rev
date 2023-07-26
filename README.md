# SteelSeries GG - Sonar API reverse engineering

Sonar is using HTTP REST API to control Sonar backend service.

On my first reverse engineering session request destinated to port 10430.
Based on previous GameSense-documentation and educated guess the port will be randomized every time the application starts.

**Port changed to 10749 after restart - confirmed**

---

C:\ProgramData\SteelSeries\GG contains some interesting files:
- coreProps.json
    - GameSense address
    - Two unknown endpoints (encrypted)
        - encryptedAddress -> Wireshark + premaster secret log
        - ggEncryptedAddress -> Wireshark + premaster secret log
- db/database.db
    - Nothing useful
- apps/sonar/db/database.db
    - Easy alternative to fetch KVP-settings / configs?

---

## Wireshark TLS decryption results

Used Wireshark with premaster secret log enabled and ...**BOOM**!

HTTP GET with TLS v1.2 to ggEncryptedAddress' /subApps responds with stuff we really need.

```json
{
    "subApps": {
        "engine": {
            "name": "engine",
            "isEnabled": true,
            "isReady": true,
            "isRunning": true,
            "shouldAutoStart": true,
            "isWindowsSupported": true,
            "isMacSupported": true,
            "toggleViaSettings": false,
            "metadata": {
                "encryptedWebServerAddress": "127.0.0.1:12261",
                "webServerAddress": "127.0.0.1:12260"
            },
            "secretMetadata": {
                "encryptedWebServerAddressCertText": "REMOVED FOR REASONS"
            }
        },
        "sonar": {
            "name": "sonar",
            "isEnabled": true,
            "isReady": false,
            "isRunning": true,
            "shouldAutoStart": true,
            "isWindowsSupported": true,
            "isMacSupported": false,
            "toggleViaSettings": true,
            "metadata": {
                "encryptedWebServerAddress": "",
                "webServerAddress": "http://localhost:12268"
            },
            "secretMetadata": {
                "encryptedWebServerAddressCertText": ""
            }
        },
        "threeDAT": {
            "name": "threeDAT",
            "isEnabled": true,
            "isReady": false,
            "isRunning": false,
            "shouldAutoStart": false,
            "isWindowsSupported": true,
            "isMacSupported": false,
            "toggleViaSettings": false,
            "metadata": {
                "encryptedWebServerAddress": "",
                "webServerAddress": ""
            },
            "secretMetadata": {
                "encryptedWebServerAddressCertText": ""
            }
        }
    }
}
```

Ok, `subApps.sonar.metadata.webServerAddress` is what we need.

Certificate is self-signed and valid for a year. Based on certificate validity times I guess it is also regenerated every runtime.

---

Just to test it again, I ran 

```
> curl https://127.0.0.1:6327/subApps -k
```
```json
{"subApps": {"engine":{"name":"engine","isEnabled":true,"isReady":true,"isRunning":true,"shouldAutoStart":true,"isWindowsSupported":true,"isMacSupported":true,"toggleViaSettings":false,"metadata":{"encryptedWebServerAddress":"127.0.0.1:12261","webServerAddress":"127.0.0.1:12260"},"secretMetadata":{"encryptedWebServerAddressCertText":"REMOVED FO REASONS"}},"sonar":{"name":"sonar","isEnabled":true,"isReady":true,"isRunning":true,"shouldAutoStart":true,"isWindowsSupported":true,"isMacSupported":false,"toggleViaSettings":true,"metadata":{"encryptedWebServerAddress":"","webServerAddress":"http://localhost:12268"},"secretMetadata":{"encryptedWebServerAddressCertText":""}},"threeDAT":{"name":"threeDAT","isEnabled":true,"isReady":false,"isRunning":false,"shouldAutoStart":false,"isWindowsSupported":true,"isMacSupported":false,"toggleViaSettings":false,"metadata":{"encryptedWebServerAddress":"","webServerAddress":""},"secretMetadata":{"encryptedWebServerAddressCertText":""}}}}
```

So this is how we get the connection information.

---

Two subfolders have some raw captures from plain HTTP traffic between Sonar & backend (`initializing/` and `volumeSettings/`).

These examples can be used to do some basic volume controlling over API.

---

To capture more just run Wireshark with port filtering
`tcp.port == ####` where `####` is port number from `subApps.sonar.metadata.webServerAddress`.

## Some examples

Mute/unmute classic-mode Game-channel
```
curl -X PUT http://127.0.0.1:13108/volumeSettings/classic/game/Mute/true -H "Content-Length: 0" -H "Host: localhost:13108"
curl -X PUT http://127.0.0.1:13108/volumeSettings/classic/game/Mute/false -H "Content-Length: 0" -H "Host: localhost:13108"
```

Set classic-mode Game-channel volume to 100%
```
curl -X PUT http://127.0.0.1:13108/volumeSettings/classic/game/Volume/1.00 -H "Content-Length: 0" -H "Host: localhost:13108"
```


## Future?

I started to reverse engineer this stuff for a plugin to Elgato Streamdeck.

Unfortunaly I am already using Elgato Wavelink, so the priority of this project has fallen really low.

I hope by releasing this information someone will see the effort to create the plugin!