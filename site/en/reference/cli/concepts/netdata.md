# Display and Manage Network Data with OT CLI

Thread Network Data contains information about Border Routers and other servers
available in the Thread network. Border Routers and devices offering services
register their information with the Leader. The Leader collects and structures
this information within the Thread Network Data and distributes the information
to all devices in the Thread Network.

Border Routers may register prefixes assigned to the Thread network and prefixes
that they offer routes for. Services may register any information relevant to
the service itself.

Border Router and service information can be stable or temporary. Stable Thread
Network Data is distributed to all devices, including Sleepy End Devices (SEDs).
Temporary Network Data is distributed to all nodes except SEDs.

## Network Data Commands

For a list of `netdata` commands, type `help`:

```
> netdata help
help
full
length
maxlength
publish
register
show
steeringdata
unpublish
Done
```

### `full` commands

The `full` commands report the flag status or resest the flag tracking whether
the "net data full" callback has been invoked.

This command requires OPENTHREAD_CONFIG_BORDER_ROUTER_SIGNAL_NETWORK_DATA_FULL.

### `length` and `maxlength` commands

The `length` command gets the current length of Thread Network Data, reported
as number of bytes. `maxlength` commands  gets the maximum observed length, or
resets the tracked maximum length.

### `publish` commands

The Network Data Publisher provides mechanisms to limit the number of similar
Service and Prefix (On-Mesh Prefix or External Route) entries in the Thread
Network Data by monitoring the network data and managing when to add or
remove entries.

The Publisher requires `OPENTHREAD_CONFIG_NETDATA_PUBLISHER_ENABLE`.

## Form network and configure prefix

1.  Generate new network configuration.

    ```
    > dataset init new
    Done
    ```

1.  Display the network configuration.

    ```
    > dataset
    Active Timestamp: 1
    Channel: 13
    Channel Mask: 0x07fff800
    Ext PAN ID: d63e8e3e495ebbc3
    Mesh Local Prefix: fd3d:b50b:f96d:722d::/64
    Network Key: dfd34f0f05cad978ec4e32b0413038ff
    Network Name: OpenThread-8f28
    PAN ID: 0x8f28
    PSKc: c23a76e98f1a6483639b1ac1271e2e27
    Security Policy: 0, onrcb
    Done
    ```

1.  Commit the new dataset to the Active Operational Dataset in non-volatile
    storage.

    ```
    > dataset commit active
    Done
    ```

1.  Enable the Thread interface

    ```
    > ifconfig up
    Done
    > thread start
    Done
    ```

1.  Display IPv6 addresses assigned to the Thread interface.

    ```
    > ipaddr
    fd3d:b50b:f96d:722d:0:ff:fe00:fc00
    fd3d:b50b:f96d:722d:0:ff:fe00:dc00
    fd3d:b50b:f96d:722d:393c:462d:e8d2:db32
    fe80:0:0:0:a40b:197f:593d:ca61
    Done
    ```

1.  Register an IPv6 prefix assigned to the Thread network.

    ```
    > prefix add fd00:dead:beef:cafe::/64 paros med
    Done
    > netdata register
    Done
    ```

1.  Display Thread Network Data.

    ```
    > netdata show
    Prefixes:
    fd00:dead:beef:cafe::/64 paros med dc00
    Routes:
    fd49:7770:7fc5:0::/64 s med 4000
    Services:
    44970 5d c000 s 4000
    44970 01 9a04b000000e10 s 4000
    Done
    ```

    Prefixes and routes include
    [argument mappings](https://openthread.io/reference/cli/) and the RLOC value.

    Service records include
    [otServiceConfig](https://openthread.io/reference/struct/ot-service-config)
    values, including `mEnterpriseNumber`, `mServiceData`,
    `otServerConfig::mServerData`, and `s` to indicate
    `otServerConfig::mStable`. The RLOC is also appended to the end of the
    record.

1.  Display the current length, in number of bytes, of Partition's Thread Network
    Data.

    ```
    > netdata length
    23
    Done
    ```
    
1.  Display IPv6 addresses assigned to the Thread interface, including the
    added prefix.

    ```
    > ipaddr
    fd00:dead:beef:cafe:4da8:5234:4aa2:4cfa
    fd3d:b50b:f96d:722d:0:ff:fe00:fc00
    fd3d:b50b:f96d:722d:0:ff:fe00:dc00
    fd3d:b50b:f96d:722d:393c:462d:e8d2:db32
    fe80:0:0:0:a40b:197f:593d:ca61
    Done
    ```

## Attach to existing network

Only the Network Key is required for a device to attach to a Thread network.

While not required, specifying the channel avoids the need to search across
multiple channels, improving both latency and efficiency of the attach process.

After a device successfully attaches to a Thread network, the device retrieves
the complete Active Operational Dataset.

1.  Create a partial Active Operational Dataset.

    ```
    > dataset networkkey dfd34f0f05cad978ec4e32b0413038ff
    Done
    > dataset commit active
    Done
    ```

1.  Enable the Thread interface.

    ```
    > ifconfig up
    Done
    > thread start
    Done
    ```

1.  After attaching to the existing network, display Thread Network Data.

    ```
    > netdata show
    Prefixes:
    fd00:dead:beef:cafe::/64 paros med dc00
    Routes:
    Services:
    Done
    ```

1.  Display the current length, in number of bytes, of Partition's Thread Network
    Data.

    ```
    > netdata length
    23
    Done
    ```

1.  Display IPv6 addresses assigned to the Thread interface.

    ```
    > ipaddr
    fd00:dead:beef:cafe:4da8:5234:4aa2:4cfa
    fd3d:b50b:f96d:722d:0:ff:fe00:fc00
    fd3d:b50b:f96d:722d:0:ff:fe00:dc00
    fd3d:b50b:f96d:722d:393c:462d:e8d2:db32
    fe80:0:0:0:a40b:197f:593d:ca61
    Done
    ```

## Debugging & diagnostics

Network Data has a limited size of 254 bytes. If Border Routers keep adding
entries (for example, prefixes, routes, or service entries) to Network Data it
can get full. When this happens, new requests from a Border Router to add new
items will be rejected or ignored by the leader. The leader does not
necessarily signal the rejection to the Border Router so the Border Router may
not immediately realize that Network Data is getting full. However, there is a
method available to detect when Network Data is getting full.

The detection method, implemented on both Border Routers and the leader, uses
a callback API mechanism and allows users to be notified when Network Data is
full. The callback can be used to take action, such as removing stale prefixes
or service entries. The `netdata full` commands are used for the flag that
tracks whether the "net data full" callback has been invoked. These commands
can report the flag's status or reset it.

For the typical use cases of Thread, it is unlikely that Network Data will get
full, even in the scenario where there are many Border Routers and they are all
adding route prefixes. 

It is technically possible for Network Data to get full, however this is often
due to misconfiguration or an issue on the Border Router. The `netdata length`
and `netdata maxlength` commands can help debug Network Data full errors.
`length` gets the current length of Network Data, reported as bytes and
`maxlength` gets the maximum observed length and can also reset the tracked
maximum length.

## License

Copyright (c) 2021-2022, The OpenThread Authors.
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:
1. Redistributions of source code must retain the above copyright
   notice, this list of conditions and the following disclaimer.
2. Redistributions in binary form must reproduce the above copyright
   notice, this list of conditions and the following disclaimer in the
   documentation and/or other materials provided with the distribution.
3. Neither the name of the copyright holder nor the
   names of its contributors may be used to endorse or promote products
   derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
POSSIBILITY OF SUCH DAMAGE.
