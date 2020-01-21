# Co-op OpenStack Training Schedule

## Monday

- Establishing accounts with the MOC
  - Adding SSH keys to keystone
- Introduction to OpenStack
- Accessing OpenStack using Horizon
  - Spawning an instance using 

## Tuesday

- Accessing OpenStack using command line tools
  - Setting API access passwords
  - Installing `python-openstackclient`

    ```
    yum -y install python-openstackclient
    ```

- Useful inspection commands

  - Getting help:

    ```
    openstack help <command>
    ```

    For example:

    ```
    openstack help server create
    ```

  - Use `openstack <noun> list` to get a list of `<nouns>`.  For example, to list servers:

    ```
    openstack server list
    ```

    Or to list networks:

    ```
    openstack network list
    ```

  - Use `openstack <noun> show <name_or_id>` to show the details about a resource. For example, so show details for a network:

    ```
    openstack network show mynet
    ```

- Spawning a Nova instance using the command line
  
  - Create a network

    ```
    openstack network create mynet
    ```

  - Create a subnet

    ```
    openstack subnet create mynet-subnet --network mynet \
      --subnet-range 192.168.20.0/24
    ```

  - Create a router

    ```
    openstack router create myrouter
    ```

  - Attach router to external network

    ```
    openstack router set myrouter --external-gateway external
    ```

  - Attach router to subnet we created earlier

    ```
    openstack router add subnet myrouter mynet-subnet
    ```

  - Create a security group

    ```
    openstack security group create mysg
    ```

  - Create security group rules to permit SSH and ICMP traffic

    ```
    openstack security group rule create mysg --protocol TCP --dst-port 22
    openstack security group rule create mysg --protocol ICMP
    ```

  - Allocate a floating ip address

    ```
    openstack floating ip create external
    ```

    Since we need to remember the ip address we allocate later on, we might want to save this in a variable. We can store the ip address in variable `$address` like this:

    ```
    address=$(openstack floating ip create external -f value -c name)
    ```

  - Create an instance

    ```
    openstack server create --key-name lars --nic net-id=mynet \
      --image centos7-1907 --flavor m1.tiny myserver
    ```

  - Associate floating ip with instance.

    We'll use that address we recorded earlier in `$address`.

    ```
    openstack server add floating ip myserver $address
    ```

  - Associate security group with instance.

    ```
    openstack server add security group myserver mysg
    ```

  Most of the above tasks only need to be done once. We can boot another server on the same network by simply repeating the `openstack server create` command with a new name. We would also need to attach a new floating ip address if the server needs one, and associate it with an appropriate security group.


### Bonus: Why Elliot was getting errors

During today's session, Elliot was unable to run the `openstack` command. We spent some time after the session debugging things and found a couple of problems.

1. User override of the Python `requests` module

    At some point, Elliot must have run `pip install --user -U requests`, or installed something else with `pip install --user` that brought in an updated `requests` module as a dependency. Because Python looks in both the system location and the user location for modules, when the OpenStack code was importing the `requests` library it was finding the updated version.

    The system `requests` module has been patched to find a list of trusted certificate-authority certificates in `/etc/pki/tls/certs/ca-bundle.crt`.  The upgraded version installed with `--user` does not have this modification, so it attempts to perform the following:

    ```
    from certifi import where
    ```

    Unfortunately, the `certifi` module was not installed, so this was causing the command to fail with a traceback.

2. Too many ebrocks

    After resolving the problem with the `requests` library, we were able to run `openstack` commands without a traceback. Unfortunately, we were __not__ able to authenticate successfully.

    It turns out there were two `ebrock@redhat.com` accounts in OpenStack:

    - One from Elliot's earlier work with the MOC
    - One created recently as part of this year's co-op

    It wasn't clear which was which. I deleted both accounts, and then we re-created one of them, after which Elliot was able to successfully authenticate.
