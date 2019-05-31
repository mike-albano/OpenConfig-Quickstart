# OpenConfig Quickstart
This is a quick & dirty example of going from a factory-defaulted
Access Point to a configured one, resulting in WiFi SSIDs being
broadcast and usable.

For a much more verbose introduction to OpenConfig, including
 modeling and code snippets, refer to this [OC by Example repo](https://github.com/mike-albano/wlpc-ocapi)

# Prerequisits
* [py_gnmicli.py](https://github.com/google/gnxi/tree/master/gnmi_cli_py)
Note, the py_gnmicli [docker container](https://github.com/google/gnxi/tree/master/gnmi_cli_py#docker)
 is also an option.
* An Access Point which supports OpenConfig (This example uses Arista)

# Step 1: Provision the AP
This uses [provision.json](./jsons/provision.json).
From the [py_gnmicli repo directory](https://github.com/google/gnxi/tree/master/gnmi_cli_py) run:
```
python ~/gnxi/gnmi_cli_py/py_gnmicli.py -m set-update -t 192.168.1.23 -p 8080 -g -o openconfig.mojonetworks.com -user admin -pass admin -x /provision-aps/provision-ap[mac=88:B1:E1:29:AE:8F]/ -val @provision.json
```
Where 192.168.1.23 is the IP of your gNMI Target (in this case the AP).

Note, if Target is using a publicly signed certificate, which is the
 case where the gNMI Target is the Cloud ap-manager, then you can drop
 the -g and -o flags.

Your output should resemble the following:
```
Performing SetRequest Update, encoding=JSON_IETF  to  192.168.1.23  with the following gNMI Path
 -------------------------
 elem {
  name: "provision-aps"
}
elem {
  name: "provision-ap"
  key {
    key: "mac"
    value: "88:B1:E1:29:AE:8F"
  }
}
 @provision.json
The SetRequest response is below
-------------------------
 response {
  path {
    elem {
      name: "provision-aps"
    }
    elem {
      name: "provision-ap"
      key {
        key: "mac"
        value: "88:B1:E1:29:AE:8F"
      }
    }
  }
  op: UPDATE
}
```
You can sanity check that your SetRequest worked by doing a GetRequest
 on that same xpath. For example:
 ```
 python ~/gnxi/gnmi_cli_py/py_gnmicli.py -m get -t 192.168.1.23 -p 8080 -g -o openconfig.mojonetworks.com -user admin -pass admin -x /provision-aps/provision-ap[mac=88:B1:E1:29:AE:8F]/
 Performing GetRequest, encoding=JSON_IETF to 192.168.1.23  with the following gNMI Path
 -------------------------
 elem {
  name: "provision-aps"
}
elem {
  name: "provision-ap"
  key {
    key: "mac"
    value: "88:B1:E1:29:AE:8F"
  }
}

The GetResponse is below
-------------------------

{
  "openconfig-ap-manager:config": {
    "country-code": "US",
    "mac": "88:B1:E1:29:AE:8F",
    "hostname": "test-01.example.net"
  },
  "openconfig-ap-manager:mac": "88:B1:E1:29:AE:8F"
}
```

# Step 2: Configure the AP
This uses [ssid_config.json](./jsons/ssid_config.json).
From the [py_gnmicli repo directory](https://github.com/google/gnxi/tree/master/gnmi_cli_py) run:
```
python ~/gnxi/gnmi_cli_py/py_gnmicli.py -m set-update -t 192.168.1.23 -p 8080 -g -o openconfig.mojonetworks.com -user admin -pass admin -x /access-points/access-point[hostname=test-01.example.net]/ -val @ssid_config.json
```
Your output should resemble the following:
```
Performing SetRequest Update, encoding=JSON_IETF  to  192.168.1.23  with the following gNMI Path
 -------------------------
 elem {
  name: "access-points"
}
elem {
  name: "access-point"
  key {
    key: "hostname"
    value: "test-01.example.net"
  }
}
 @ssid_config.json
The SetRequest response is below
-------------------------
 response {
  path {
    elem {
      name: "access-points"
    }
    elem {
      name: "access-point"
      key {
        key: "hostname"
        value: "test-01.example.net"
      }
    }
  }
  op: UPDATE
}
```
Again, you can sanity check that your SetRequest worked by doing a GetRequest
 on that same xpath. For example:
```
python ~/gnxi/gnmi_cli_py/py_gnmicli.py -m get -t 192.168.1.23 -p 8080 -g -o openconfig.mojonetworks.com -user admin -pass admin -x /access-points/access-point[hostname=test-01.example.net]/
Performing GetRequest, encoding=JSON_IETF to 192.168.1.23  with the following gNMI Path
 -------------------------
 elem {
  name: "access-points"
}
elem {
  name: "access-point"
  key {
    key: "hostname"
    value: "test-01.example.net"
  }
}

The GetResponse is below
-------------------------

{
  "openconfig-access-points:hostname": "test-01.example.net",
  "openconfig-access-points:system": {
    "cpus": {
      "cpu": [
        {
          "index": "ALL",
          "state": {
            "index": "ALL",
            "total": {
              "instant": 13,
              "min": 8,
              "max": 78,
              "interval": "120",
              "min-time": "1559323384",
              "max-time": "1559323374",
*SNIP*
```
You should limit whats being returned in the GetResponse using xpaths.
 For example:
```
python ~/gnxi/gnmi_cli_py/py_gnmicli.py -m get -t 192.168.1.23 -p 8080 -g -o openconfig.mojonetworks.com -user admin -pass admin -x /access-points/access-point[hostname=test-01.example.net]/ssids/ssid[name=student1_open]/config/opmode
```
You should now see your SSIDs, and be able to connect to them.
