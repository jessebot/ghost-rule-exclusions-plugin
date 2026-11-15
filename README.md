# OWASP CRS - Ghost Exclusions Plugin

This is a first attempt at creating an exclusions plugin for Ghost using the template repo provided by the official OWASP CSR team. This will likely be mirrored to codeberg.org eventually but is also available at [git.open.engineering/oeng/ghost-exclusions-plugin](https://git.open.engineering/oeng/ghost-exclusions-plugin) as the author is rarely on GitHub these days.

![Integration tests](https://github.com/coreruleset/template-plugin/actions/workflows/integration.yml/badge.svg)

The CRS plugin documentation can be found on the [website](https://coreruleset.org/docs/configuring/plugins/).

## How does this Template Plugin work?

The Template Plugin has two active rules and three rules that are commented
out:

Rule 9500010 in template-config.conf checks for the existence of the
TX variable `template-plugin_enabled`. If the variable is not set,
it is being created and set to 0.
The rule is commented out. That means the plugin is enabled
by default. Uncomment this rule to disable the plugin.
This rule is mandatory for CRS plugins.

Rule 9500020 in template-config.conf sets the TX variable `template-plugin_foobar`
to 1. It is commented out. Uncomment it to set the variable.

Rule 9500100 in template-config.conf checks for the request parameter
template-plugin-test-parameter. If is present and not empty, then
a rule alert is triggered and the anomaly score is raised with
a severity CRITICAL score.

Rule 9500099 in template-before.conf checks for the
TX variable `template-plugin_enabled`. If the variable is existing and it is
set to 0, then the subsequent rules of the plugin are removed for the
current transaction. This effectively disables the plugin for the
current transaction.
This rule is mandatory for CRS plugins. When writing your own plugins,
it is important to make sure the control statement in this rule removes
the right rule range.

Rule 9500100 in template-before.conf checks for the existence of the
request parameter `template-plugin-test-parameter`. If the parameter exists
and is non-empty, a critical alert is thrown.

## How to adopt the Template Plugin to your needs?

Feel free to take this template plugin and adopt it to your needs.

The following steps are necessary to create a new plugin that
follows the established standard for CRS plugins:

* Make sure the control statement in 9500099 is correct.
* Test your rule set throughly, namely the interaction with CRS.
* Publish your plugin rule set

## Order of rules with plugins and existing CRS rules

The rules run in the order described in the following sections.

### Phase 1

As you know, ModSecurity rules can run in multiple phases. The
engine will first execute all rules in phase 1, then all the
rules in phase 2. If a phase 2 rule is listed in a rules file
before a phase 1 rule, then the phase 1 rule is executed first.

* modsec-recommended-rules (rule id 200000 and friends)
* CRS configuration based on rules in crs-setup.conf
* plugins *-config-before.conf
* plugins *-before.conf
* CRS init rules in REQUEST-901-INITIALIZATION.conf
* CRS rules running in phase 1
* CRS rule 949111: blocking early
* plugins *-after.conf

Please note that plugin rules in the *-before.conf file will
run before the formal initialization of CRS in the 901 rules
file. If you do anomaly scoring in phase 1 before the
901 rules are executed, the scores will be overwritten.

If you want to score in phase 1, you need to put your rules
in the plugin *-after.conf file. Yet the rules in this file
are being executed after the early blocking is being executed.

In conclusion: Phase 1 anomaly scoring via plugins and early
blocking are incompatible. This is a weakness of
the CRS plugin architecture that is very hard to improve.


## Testing

Plugins can also be shipped with their own tests. Any `ftw` test yaml file that you add below the `tests/regression` directory will be run by the integrated pipeline.
We suggest you to use a new subdirectory for your specific plugin's test, e.g. `tests/regression/<your-plugin>`, and create one file per rule you want to test.

Many plugins target rule exclusions, so it will be rather common to see tests matching a REQUEST_URI but matching for `no_log_contains`, as we want to test that we are not being blocked.

## License

Copyright (c) 2021-2022 OWASP CRS project. All rights reserved.

The OWASP CRS and its official plugins are
distributed under Apache Software License (ASL) version 2.
Please see the enclosed LICENSE file for full details.
