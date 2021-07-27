# THE RESOURCE MODULE DOC

## Code is like humor. When you have to explain it,it’s bad. ~ _Random Motivation_

## Is your model ready?

```
gitclonehttps://github.com/ansible-network/resource_module_models.git
```
Once the above repo is cloned go to the desired collectionand add your model there!! And talk to the
team to get it _approved_. YAML syntax helplink,link

## Let’s get started with the Resource Module development!!!

At first, we would need a builder for our whole codebaseto get scaffolded from a tool -

Grab it from here -https://github.com/ansible-network/cli_rm_builder

```
gitclonehttps://github.com/ansible-network/cli_rm_builder.git
```
This is how your collection _dir_ should look like,follow the Github _namespaces_ for anything other thanwhat
specified below and fork them accordingly ...

```
~/{Something}/{Anything}/collections
❯ tree -L 3
.
└── ansible_collections
├── ansible
│ ├── netcommon
│ └── utils
├── ansible_network
│ ├──cli_rm_builder
│ ├── collection_prep
│ └── network-image-builder
├── cisco
│ ├── ios
│ └── nxos
└── vyos
└──vyos
```
Once the cli_rm_builder repo is cloned in the desiredlocation and you have your target
collection also cloned at the apt place. So, I wouldwalk you through developing aLogging
moduleforVyosplatform. _[forked and switched tothe desired branch]_
Now we are ready to use the tool to generate the base-levelcode with which we proceed.


I am adding my playbooks to the~/{Something}/{Anything}level feel free to change it.

```
❯catrm_builder_run.yml
---
```
- hosts:localhost
    gather_facts:yes
    roles:
       - ansible_network.cli_rm_builder.run
    vars:
       docstring:
/home/{repos}/resource_module_models/models/vyos/logging_global/vyos_logging_global.yaml
    rm_dest:/home/{user}/{S}/{A}/collections/ansible_collections/vyos/vyos
    resource:logging_global ##### name of the module
    collection_org:vyos
    collection_name:vyos
    ansible_connection:local

```
❯ ansible-playbook rm_builder_run.yml
```
Post execution of the above command there should befew new files in your branch ready for
the development of the resource module.

# Important links at this point -

* A glance of RMs -link,link2,link3oh! You don’tlike to read? Watch this -link,link
* Debugging Ansible Network Modules with VSCode -link
* Setup Python Virtual-env for Development -link,link
* In case you are brave enough to do a local setupof Network device in PSI -link

# Introduction to RM files -

Looking at the collection where you are adding yourresource module we should see new files
added -

vyos_logging_global.py<- the entry point to the resource _module_ code and the module
documentation. To change or update any argspec attributeduring development just update it
here in the docstring and then rerun therm_builder_run.ymlwith docstring under the vars
commented out.

```
../collections/ansible_collections/vyos/vyos/plugins/modules/vyos_logging_global.py
```

logging_global.py<- under _facts_ directory. The code in this file is responsible for converting
native on-box configuration to Ansible structured data as per the argspec of the module by using
the list of parsers defined in rm_templates/logging_global.py.The on-box config can either be
fetched from the target device or from the value setin the `running_config` key in the task when
the module is run with `state: parsed`. The call toget data from the target device is often
wrapped within a method. This is done to facilitateeasier mocking when writing unit tests.

```
../collections/ansible_collections/vyos/vyos/plugins/module_utils/network/vyos/facts/logging
_global/logging_global.py
```
facts.py<- You need to manually append the existenceof your new resource module’s Facts
class to the `FACTS_RESOURCE_SUBSET` dictionary inrthis file for facts to be generated as
the call for facts is from a common instance i.e netcommon
The import -

```
from
ansible_collections.vyos.vyos.plugins.module_utils.network.vyos.facts.logging_global.logging
_globalimport (
Logging_globalFacts,
)
```
The entry under global Var -

```
FACT_RESOURCE_SUBSETS = dict(logging_global=Logging_globalFacts,)
```
```
../collections/ansible_collections/vyos/vyos/plugins/module_utils/network/vyos/facts/facts.p
y
```
logging_global.py<- The _argspec_ file is the pythonlevel representation of your model.
You would never need to edit it manually, change themodel in the module file and
cli_rm_builder should take care of updating it.

```
../collections/ansible_collections/vyos/vyos/plugins/module_utils/network/vyos/argspec/loggi
ng_global/logging_global.py
```
logging_global.py<- the _rm_template_ is one of themost vital components of a Resource
Module since the conversion of native on-box configurationto structured data and vice-versa is
facilitated by the parser templates that are definedin this file. Time to spin up regex/ jinja
templating skills for this file. The better they arethe easier it would be to get a higher score in
Unit Test Coverage (UTC).
Links:regexParser1,RegexParser2,Jinja2Parser,AnsibleJ2Tester

```
../collections/ansible_collections/vyos/vyos/plugins/module_utils/network/vyos/rm_templates/
logging_global.py
```

Logging_global.py<- the _config_ file contains allthe core logic of how the execution should
behave in various states. Be creative here!
In here you get _want_ [the playbook] and _have_ [theconfig that came from the facts rendered]
All the states and their comparison logic goes here.

```
/home/sagpaul/Work/bannerNconfig/collections/ansible_collections/vyos/vyos/plugins/module_ut
ils/network/vyos/argspec/logging_global/logging_global.py
```
And, You might see a couple of __init__.py files generated,required for Ansible tests to pass!

## PHASE - 1 Gathering facts from the target device

The first step in building a resource module is towrite facts code that converts device native
configuration to structured data. This is done bycomparing the device config with a set of
pre-defined “Parser Templates” that define regexesto parse the native config.

Both the list of templates and the config are fedto an object of the NetworkTemplate class, on
which the `parse()` method is then invoked. The outputof the `parse()` method is
semi-structured data that might need some additionalupdates to match the module’s argspec
format.

Before finally rendering this data as facts, it isvalidated against the module’s argspec by the
validate_config() method which fails if the data doesnot match the schema defined in the
argspec.

## Anatomy of a Parser Template:

name-
The name or unique identifier of the parser template.

getval-
A regular expression using named capture groups tostore the extracted data.Here
goes a regex that can break a command or a part ofthe command that is read from the device
config and this contributes to generating facts fromdevice native config. NOTE- This regex is
not for validation, as this will not be used whileforming the commands so we can just use
simple regex forms to fragment the command and assignit to variables to use while we make
our facts.

setval-
This is used to generate device native config fromAnsible structured data. It can either
be a Python function or a Jinja2 template works.


result-
A data tree, populated as a template, from the parseddata.This is where we generate
the facts with the help of variables that are formedvia the regex fragmentation we did in _getval_ It
may match the part of facts the whole parser is writtenfor i.e argspec i.e model

remval-
(Optional) This is used to specify a command to negatean attribute. It can either be a
Python function or a Jinja2 template. This is usuallynot needed, as in most cases simply
negating the command generated by setval does thejob.
However, this comes in handy when the command to removean attribute is significantly
different from ‘setval’.

compile-
(Optional) This is to be used for a complex modelwhere parsers are broken down into
multiple ones and are then referenced like namespaces.By default, the name of the parser itself
is used to extract an attribute for want and havedictionaries.
For example:

want = {'k1': {'k2': {'k3': 'newval', ‘k4’: 'anotherval'}}}
have = {'k1': {'k2': {'k3': 'oldval', ‘k4’: 'anotherval'}}}

With parser name 'k1.k2.k3', the RMEngineBase willextract the value of the nested key 'k3'
from both want and have and then compare it in orderto decide if an update is required. In this
case, it will compare 'k3': 'newval' and 'k3': 'oldval'.However, if the parser template has
`compval: k1.k2` defined, the value of the key `k2`(which is a dictionary itself) will be used. So
here, the comparison will happen between 'k2': {'k3':'newval', k4: 'anotherval' } and {'k3': 'oldval',
k4: 'anotherval'}

shared-
(Optional) The shared key makes the parsed valuesavailable to the rest of the parser
entries until matched again.This enables the data/resultof the parser to be shared among other
parsers for reuse.

Example parsers -

```
vyos@vyos:~$ show configuration commands | grep syslog
set system syslog console facility all
set system syslog console facility local7 level 'err'
set system syslog console facility news level 'debug'
```
Given the set of commands the parser _can_ look like- the model for which looks like -link


### PARSERS = [

### {

```
"name": "console",
"getval": re.compile(
r"""
^set\ssystem\ssyslog\sconsole
(\sfacility\s(?P<facility>news|all|local[0-7]))?
(\slevel\s(?P<level>'(err|notice|info|debug|all)'))?
$""", re.VERBOSE),
"setval": console_cmd,
"result": {
"config": {
"console_params": [{
"facility": "{{ facility }}",
"level": "{{ level }}",
}]
}
},
},
```
Having setval ready at this point is not required,we can start off by executing our first playbook

### ---

- name: check GATHEREDstate
    hosts: your.host.name
    gather_facts: no
    tasks:
    - name: Gather logging config
       vyos.vyos.vyos_logging_global:
          state: gathered

Note-
If there is a _list of items_ in the generated facts, it is suggested to sort them before they are
rendered, in order to to get consistent output acrossdifferent Python versions. This also helps
with assertions while working on Unit or IntegrationTests.
Or

### ---

- name: check PARSED state
    hosts: your.host.name
    gather_facts: no


```
tasks:
```
- name: Parse the provided configuration
    register: result
    vyos.vyos.vyos_logging_global:
       running_config:"{{ lookup('file', 'raw_vyos.cfg') }}"
       state: parsed

```
raw_vyos.cfg
```
This is just a flat file that holds the config thatwe parse after the show commands are executed
on the target device.

You should have a working facts code as of now! Andthe _gathered/parsed_ state should work
before you proceed further.

### PHASE 1.1 - THE RM AESTHETICS

Unlike all other modules, the RM is specifically madeto deliver a more genuine outlook of the
operations that it performed i.e your get to see moreinformation about the play post the
execution of a resource module. On execution of analready made resource module, you should
see few parts on the output for that just add _-vvvv_ [verbose out],

There you might see -

after-> The configuration/ facts after a playbook’stasks/operations are performed.
before-> The configuration/ facts before the playbook’stask/operations are performed.
commands-> The set of commands that the RM executionspit out on basis of the state used,
are the only set of commands that are sent to thedestination device/target nodes.
invocation-> The raw representation of how the playbookis considered by the RM in terms of
facts.

Example -
THE Play -

```
---
```
- name: Example
    hosts: devnetXe
    gather_facts: no
    tasks:
       - name: Apply commands for provided configuration
          cisco.ios.ios_logging_global:


```
config:
buffered:
severity: notifications
size: 5099
xml: true
hosts:
```
- hostname: 10.0.1.
state: merged
####check_mode: True<- use check mode to not apply the commands

THE Output -

```
changed: [sandbox-iosxe-latest-1.cisco.com] => {
"after": { <- The config we pushed via our playbook
"buffered": {
"severity": "notifications",
"size": 5099 ,
"xml": true
},
"hosts": [
{
"hostname": "10.0.1.12"
}
]
},
"before": {}, <- There was no config before
"changed": true, <- Tells you did anything change?
"commands": [ <- The commands that are sent to the device
"logging host 10.0.1.12",
"logging buffered xml 5099 notifications"
],
"invocation": { <- The play agaist the whole facts skeleton
"module_args":{
"config": {
"buffered": {
"discriminator": null,
"filtered": null,
"severity": "notifications",
"size": 5099 ,
"xml": true
},
"buginf": null,
```

### ....

### ....

```
"history": null,
"hosts": [
{
"discriminator": null,
"filtered": null,
"hostname": "10.0.1.12",
"ipv6": null,
"sequence_num_session": null,
"session_id": null,
"stream": null,
"transport": null,
"vrf": null,
"xml": null
}
],
"logging_on": null,
"snmp_trap": null,
"source_interface": null,
"trap": null,
"userinfo": null
},
"running_config": null,
"state": "merged"
}
}
}
```
## PHASE - 2 THE CONFIG / MERGED and other STATEs

Let’s talk about the different states! -link, again!you hate reading watchlink

_WANT_ is basically the exact representation of thetask, which is accessible in the config code.

_HAVE_ is the on-box config as structured data thatis rendered by the facts code
_OP_ is just a representation of the commands that areformed based on want/have and states


MERGED- don’t delete anything just add things not in have from want

```
WANT HAVE OP Comment
```
```
{A, B, C, D} {A,B,E} {C,D} Changed
```
```
{} {A,B,C,D,E} {} No change
```
```
{A, B, C, D} {} {A, B, C, D} Changed
```
REPLACED- replace similar data from the data in wantkind of an update operation, don’t touch
data that is not having a like counterpart in have. _rA <- replace A_

```
WANT HAVE OP Comment
```
```
{A, B, C, D} {A, B, E, F} {rA, rB, E, F} Changed A, B
```
OVERRIDDEN- Change everything as per your want andnegate everything from have.
_nA - Negate A_

```
WANT HAVE OP Comment
```
```
{A, B, C, D} {A,B,E} {A, B, nE, D} Changed
```
```
{} {A,B,C,D,E} {} No change
```
```
{A, B, C, D} {} {A, B, C, D} Changed
```
```
{A, B, C, D} {E,F,G} {nE,nF,nG,A, B, C, D} Changed
```
DELETED- Negate everything in have if anything specifiedin want deletes that portion from have

```
WANT HAVE OP Comment
```
```
{A, B, C, D} {A,B,E} {nA, nB} Changed
```
```
{} {A,B,C,D,E} {nA,nB,nC,nD,nE} Changed
```
```
{A, B, C, D} {} {} No change
```
```
{A, B, C, D} {E,F,G} {} No change
```
RENDERED- Pass in a config with the rendered stateit is supposed to tell you all the set of
commands that would be formed on the supplied config(without actually connecting to the
target device), it is different from check mode. Docheck it out.


PARSED- The parsed state is just opposite to therendered state it tells you how the
invocation/facts would look like when you supply therunning_config/ raw config from a device.

Some important links at this point-

- Read about parsed -link
- Command op / prompts -link
- Debugging network Modules -link

On the config side code, you get a lot of creativeliberty to handle the _want_ and _have,_ compare
them and make them work as per the states. The finalset of commands that need to be
executed on the target nodes are formed here, basedon the _setval_ values defined within the
parsers.
Implementation of _list to dict_ on every attributeis imp on the entry point of config code before it
starts getting processed on the basis of states. Asthe _compare() method understands it better.

A comparison of two dictionary of dictionaries iseasier and more efficient than a comparison of
two lists of dictionaries. Hence, to optimally leveragethe RMEngineBase, it is important that we
convert all lists to dicts to dicts of dicts beforestarting with the comparison process. Refer the
following as example:

```
● https://github.com/ansible-collections/cisco.nxos/blob/main/plugins/module_utils/network/
nxos/config/route_maps/route_maps.py#L190-L
● https://github.com/ansible-collections/arista.eos/blob/main/plugins/module_utils/network/
eos/config/bgp_global/bgp_global.py#L365-L
● https://github.com/ansible-collections/cisco.iosxr/blob/main/plugins/module_utils/network/
iosxr/config/bgp_global/bgp_global.py#L376-L
```
With the config development in place, there are fewthings to keep a note of to make the code
clean and reusable by the rest of the modules withinthe same platform,

```
..collections/ansible_collections/{{platform}}/{{platform}}/plugins/module_
utils/network/{{platform}}/utils/utils.py
```
At the above path, utils.py creates a set of definedmethods that includes flattening the config or
processing the list_to_dict operations for that platform.Adding a generic method here and
making the whole module reuse that existing code addsup to the code quality.

Back to config, how the _setvals_ are picked up afterthe compare method is at a point to generate
the commands that are finally used to apply the necessarycomparison.


So, the compare method on comparison of two dicts refers to it by the name of the parsers and
tries to match that with the defined list of parsersin the config code. Adding the parser names in
namespace format helps the compare method to reducethe namespace based on the
dictionary it is looking at and does the setval computationon the basis of that. There is no direct
relation between the _result_ key and _setval_ key inthe parsers. The data available at setvals to
generate the command may or may not be aligned withthe facts/results from parsers. It
depends on the flattening logic written in configwhich molds our have and want data or better I
say wantd and haved to be easily compared. Here ifa dict contains

```
haved = { ‘key1’ : { ‘key2’ : 100 , }}
wanted = { ‘key1’ : { ‘key2’ : 10 , }}
```
A parser namedSomethingwill do the compare and pushthe whole dict for being processed in
setvals. Whereas a parser namedSomething.Anythingwill do the comparison on a level
under Something as per the above example.

## PHASE - 3 Unit Test and Integration Tests

### #TODO

### #TODO


```
>>> from module root <<<
tox -e black.
tox -elinters.
```
```
>>> for integration test run on a single module <<<
ansible-test network-integration --python 3.5 --inventory
/home/{something}/MyVnvs/development/dev/inventory_lite ios_logging_global
```
- vvv

```
>>> sanity with specific python <<<
ansible-test sanity --python 3.
ansible-test sanity --python 3.5 --test=validate-modules
```
```
>>> for unit test run on a single module <<<
ansible-test units --python 3.
tests/unit/modules/network/ios/test_ios_logging_global.py -vvvv
ansible-test coverage erase
ansible-test units --python 3.5 --coverage
ansible-test coverage report
ansible-test coverage html
```
```
>>> run black formatting on a single line <<<
black -l 279
plugins/module_utils/network/ios/rm_templates/logging_global.py
```
The game I play often -Command line heros


