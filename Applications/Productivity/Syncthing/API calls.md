---
created: 2024-04-02T09:53
updated: 2024-04-02T10:06
tags:
  - Syncthing
  - API
related link: https://docs.syncthing.net/dev/rest.html
---
> [!info] Syncthing exposes a REST interface over HTTP on the GUI port.
> This is used by the GUI (from Javascript) and can be used by other processes wishing to control Syncthing. In most cases both the input and output data is in JSON format. **The interface is subject to change.**

## API Key

To use the REST API, an API key must be set and used. 

The API key can be generated in the GUI, or set in the `configuration/gui/apikey` element in the configuration file. 

To use an API key, set the request header `X-API-Key` to the API key value, or set it as a `Bearer` token in the `Authorization` header. For example :

```
curl -X GET -H "X-API-Key: abc123" http://localhost:8384/rest/...
```

```
curl -X GET -H "Authorization: Bearer abc123" http://localhost:8384/rest/...
```

> [!warning] Add `-k` flag when using HTTPS with a Syncthing generated or self signed certificate.

One exception to this requirement is `/rest/noauth`. you do not need an API key to use those endpoints. This way third-party devices and services can do simple calls that don’t expose sensitive information without having to expose your API key.

## Further examples

Adding / Accepting a device :
```http
curl -X PUT http://localhost:8384/rest/config/devices \
-H "X-API-Key: [REDACTED]" \
-H 'Content-Type: application/json' \
-d '[ { "deviceID": "[REDACTED]", "name": "Tom-Thinkpad", "addresses": [ "dynamic" ], "paused": false } ]' 
```

Create and share a folder: 
```http
curl -X PUT http://localhost:8384/rest/config/folders \
-H "X-API-Key: [REDACTED]" \
-H 'Content-Type: application/json' \
-d '[ { "id": "tomThinpad-test", "name": "Tom-Thinkpad_test", "path": "/mnt/pve/local-nvme/bindmount-tests/", "devices": [ { "deviceID":"[REDACTED]" } ] } ]' 
```

Modify a share to "Ignore Permissions" :
```http
curl -X PATCH http://localhost:8384/rest/config/folders/tomThinpad-test \
-H "X-API-Key: [REDACTED]" \
-H 'Content-Type: application/json' \
-d '[ { "ignorePerms": false } ]' 
```
