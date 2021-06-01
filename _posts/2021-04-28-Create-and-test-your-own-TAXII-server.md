---
title: "Create and test your own TAXII server"
layout: post
date: 2021-04-28 15:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Threat Intelligence
category: blog
author: coreflood
description: ~.~
---

Since there's almost no blog posts talking about how to set up your own TAXII server i thought this could help.

> Trusted Automated Exchange of Intelligence Information (TAXII™) is an application protocol for exchanging CTI over HTTPS. ​TAXII defines a RESTful API (a set of services and message exchanges) and a set of requirements for TAXII Clients and Servers
> 
> TAXII was specifically designed to support the exchange of CTI represented in STIX

I'll be using [EclecticIQ OpenTAXII](https://github.com/eclecticiq/OpenTAXII) in docker and [Cabby](https://github.com/eclecticiq/cabby) for testing it.

Clone the repo because we're going to be changing some configurations instead of running the `eclecticiq/opentaxii` image from docker hub

`git clone https://github.com/eclecticiq/OpenTAXII.git` 

# Modify configuration files

First we need to edit `examples/data-configuration.yml`

Here you can edit the username and password of the default user

> **NOTE**: if you add a new user using the `opentaxii-create-account` command from inside the container, it won't take effect in the database.
You'll have to rebuild the image with the new user added to `data-configutation.yml` or add the user directly in the database. 
to access the database execute `sqlite3 /data/auth.db` from inside the container.


I set  `authentication_required` of `inbox_a` to `yes` to test server authentication later

Here's the edited part of `data-configuration.yml`:
```yaml
services:
  - id: inbox_a
    type: inbox
    address: /services/inbox-a
    description: Custom Inbox Service Description A
    destination_collection_required: yes
    accept_all_content: yes
    authentication_required: yes
```

Now save and let's edit the `Dockerfile` to copy `data-configuration.yml` into the container

in the Dockerfile add this line under the `ENTRYPOINT` command
```buildoutcfg
COPY  examples/data-configuration.yml /data-configuration.yml	
```

# Fix authentication error


if you build and run the server now you'll probably get  ```401 Unauthorized```  (unless they fixed it in their github repo)

basically because opentaxii is treating your token as bytes and it's in fact a string
so let's fix that

we need to edit ```opentaxii/management.py``` (this is where the authentication error is coming from)

Replace this line

```python
return jsonify(token=token.decode('utf-8'))
```

with this:
```python
return jsonify(token=token)
```


# Build and run the server

Now that we have fixed everything, build and run the image:

```buildoutcfg
 docker build --no-cache  --network=host -t opentaxii  -f Dockerfile .
 docker run -d --network=host opentaxii
```

according to the documentation now the server *should* be available on `localhost:9000` 

To check run this command 
```bash
 curl -d 'username=admin&password=admin' http://localhost:9000/management/auth
```

If everything's good you should get a token and continue to the [Testing the server](#testing-the-server) section , if not or if you need to see the server logs

Go into the container:

```bash
 docker exec -ti <container-id> bin/bash
```

you can get the container id using ```docker ps```

Rerun the server using a different port:

```bash
gunicorn opentaxii.http:app --bind localhost:1234
```
To test it, from your local computer run:

```bash
curl -d 'username=admin&password=admin' http://localhost:1234/management/auth
```
you should get the token now.


# Test the server

Now that the server's up and running, here's the python script to test it

```python
from cabby import create_client
client = create_client('localhost',
                        use_https = False,
                        port = '1234',
                        discovery_path='/services/discovery-a')
client.set_auth(
    username='admin',
    password='admin',
    # URL used to obtain JWT token
    jwt_auth_url='/management/auth'
)
# Check the available services to make sure inbox service is there
services = client.discover_services()
print(f"Services: {services}")
# Get the data that we want to send
with open("examples/stix/stuxnet.stix.xml") as stix_file:
    stix_data = stix_file.read()
binding = 'urn:stix.mitre.org:xml:1.1.1'
# URI is the path to the inbox service we want to use in the taxii server
client.push(stix_data, binding,
            collection_names=['collection-a'],
            uri='/services/inbox-a')
print(f"Successfully exported to TAXII server.")
```
save this in the `OpenTAXII` dir and run, now you should see the services printed.

To verify that the data is pushed, run this in your local computer:

```taxii-poll --path http://localhost:1234/services/poll-a -c collection-a --username admin --password admin```

you should get the contents of `examples/stix/stuxnet.stix.xml`












