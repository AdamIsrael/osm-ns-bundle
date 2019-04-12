# Prototype for Network Service Primitives

This bundle is composed of three charms:
- ns, using the experimental `osm-ns` base layer, which allows a Network Service charm to orchestrate action execution across charms within a model.
- vnf-user, which mimics a user database
- vnf-policy, which mimics a policy management database


## Use-case

Consider charms that individually manage a user and policy database.

In order to add a subscriber, you need to add the user and then set policy. You could manually execute the actions, but to do so in an automated fashion you need something that understands how to talk to both charms.

Enter the concept of a "Network Service" charm.


# Workflow

The `ns` charm exposes an action named `add-user` that accepts four parameters:
- username
- bw
- qos
- tariff

The `vnf-user` charm exposes an action named `add-user` that accepts two parameters:
- username
- tariff

The `vnf-policy` charm exposes an action named `set-policy` that accepts three parameters:
- user_id
- bw
- qos


When `ns.add-user` is executed, it will first call `vnf-user.add-user`, passing `username` and `tariff`. If successful, this will return the ID of the user.

Next, the `ns` charm will execute `vnf-policy.set-policy`, passing `user_id`, `bw`, and `qos`.

If either action fails, `ns.add-user` will fail. Otherwise, it will return successfully.

### Example

Once the three charms are deployed to Juju, you can use the below commands to manually execute the NS primitive, which is contained by the ns charm.

```bash

juju add-model ns-demo
juju deploy cs:~aisrael/osm-ns-bundle/bundle.yaml

# Run the NS primitive
$ juju run-action ns/0 add-user username=adam tariff=1 bw=2 qos=3
Action queued with id: 691dbfd6-46c5-4d21-8a7c-fbd3849e33a5

# Check the status of the primitive. Once it's complete we can view its output.
$ juju show-action-status 691dbfd6-46c5-4d21-8a7c-fbd3849e33a5
actions:
- action: add-user
  completed at: "2019-02-05 09:50:56"
  id: 691dbfd6-46c5-4d21-8a7c-fbd3849e33a5
  status: completed
  unit: ns/1

# The output of the primitive contains information provided by the NS charm, which it received from the VNF charms.

$ juju show-action-output 691dbfd6-46c5-4d21-8a7c-fbd3849e33a5
results:
  policy-set: '{''updated'': ''True''}'
  user-id: "63"
status: completed
  timing:
    completed: 2019-02-05 09:50:56 +0000 UTC
    enqueued: 2019-02-05 09:50:26 +0000 UTC
    started: 2019-02-05 09:50:29 +0000 UTC

```

# Known Limitations and Issues

Because the Network Service charm maps to specific actions within the other charms, the NS charm is not very robust. It knows how to execute specific actions, and does not currently support a way to dynamically learn or be informed of which actions to call for which function.