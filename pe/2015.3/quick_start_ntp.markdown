---
layout: default
title: "NTP Quick Start Guide"
canonical: "/pe/latest/quick_start_ntp.html"
---

[downloads]: http://info.puppetlabs.com/download-pe.html
[sys_req]: ./install_system_requirements.html
[agent_install]: ./install_agents.html
[install_overview]: ./install_basic.html

Welcome to the Puppet Enterprise NTP Quick Start Guide. This document provides instructions for getting started managing an NTP service using the Puppet Labs NTP module.

The clocks on your servers are not inherently accurate. They need to synchronize with something to let them know what the right time is. NTP is a protocol designed to synchronize the clocks of computers over a network. NTP uses Coordinated Universal Time (UTC) to synchronize computer clock times to within a millisecond.

Your entire datacenter, from the network to the applications, depends on accurate time for many different things, such as security services, certificate validation, and file sharing across nodes.

NTP is one of the most crucial, yet easiest, services to configure and manage with Puppet Enterprise. Using the Puppet Labs NTP module, you can do the following tasks:

* Ensure time is correctly synced across all the servers in your infrastructure.
* Ensure time is correctly synced across your configuration management tools.
*  Roll out updates quickly if you need to change or specify your own internal NTP server pool.

This guide will step you through the following tasks:

* [Install the `puppetlabs-ntp` module](#install-the-puppetlabs-ntp-module).
* [Use the PE console to add classes from the NTP module to your agent nodes](#use-the-pe-console-to-add-classes-from-the-ntp-module).
* [Use the PE console Events page to view changes to your infrastructure made by the main NTP class](#use-the-events-page-to-view-changes-made-by-the-ntp-class).
* [Use the PE console to edit parameters of the main NTP class](#use-the-pe-console-to-edit-parameters-of-the-ntp-class).


## Install Puppet Enterprise and the Puppet Enterprise Agent

If you haven't already done so, install PE. See the [system requirements][sys_req] for supported platforms.

1. [Download and verify the appropriate tarball][downloads].
2. Refer to the [installation overview][install_overview] to determine how you want to install PE, and follow the instructions provided.
3. Refer to the [agent installation instructions][agent_install] to determine how you want to install your PE agents, and follow the instructions provided.

>**Note**: You can add the NTP service to as many agents as needed. For ease of explanation, our console images and instructions might show only one agent.


## Install the puppetlabs-ntp Module

The puppetlabs-ntp module is part of the PE [supported modules](http://forge.puppetlabs.com/supported) program. These modules are supported, tested, and maintained by Puppet Labs. You can learn more about the puppetlabs-ntp module by visiting [http://forge.puppetlabs.com/puppetlabs/ntp](http://forge.puppetlabs.com/puppetlabs/ntp).

**To install the puppetlabs-ntp module**:

From the PE master, run `puppet module install puppetlabs-ntp`.

You should see output similar to the following:

        Preparing to install into /etc/puppetlabs/code/environments/production/modules ...
        Notice: Downloading from http://forgeapi.puppetlabs.com ...
        Notice: Installing -- do not interrupt ...
        /etc/puppetlabs/code/environments/production/modules
        └── puppetlabs-ntp (v3.1.2)

> That's it! You've just installed the puppetlabs-ntp module. You'll need to wait a short time for the Puppet server to refresh before the classes are available to add to your agent nodes.

## Use the PE Console to Add Classes from the NTP Module

The NTP module contains several **classes**. [Classes](/puppet/4.3/reference/lang_classes.html) are named chunks of Puppet code and are the primary means by which Puppet Enterprise configures nodes. The NTP module contains the following classes:

* `ntp`: the main class; this class includes all other classes (including the classes in this list).
* `ntp::install`: this class handles the installation packages.
* `ntp::config`: this class handles the configuration file.
* `ntp::service`: this class handles the service.

We're going to add the `ntp` class to a node group we'll create, called **NTP**, which will contain all of your nodes. Depending on your needs or infrastructure, you may have a different group that you'll assign NTP to, but these same instructions would apply.

**To create the NTP node group**:

1. In the console, click __Nodes__ in the side navigation bar and select __Classification__.
2. In the **Node group name** field, name your group **NTP**.
3. Click **Add group**.

   **Note**: Leave the **Parent name** and **Environment** values as their defaults (**default** and **production**, respectively).

4. On the __Classification__ page, select the __NTP__ group, and click the __Rules_ tab.
5. In the **Fact** field, enter `name`.
6. From the **Operator** drop-down list, select **matches regex**.
7. In the **Value** field, enter `.*`.
8. Click **Add rule**.

   This rule will ["dynamically" pin all nodes]((./console_classes_groups.html#adding-nodes-dynamically) to the **NTP** group. Note that this rule is for testing purposes and that decisions about pinning nodes to groups in a production environment will vary from user to user.

**To add the** `ntp` **class to the NTP group**:

1. On the __Classification__ page, select the __NTP__ group.

2. Click the __Classes__ tab.

3. In the __Class name__ field, begin typing `ntp`, and select it from the autocomplete list.

   **Tip**: You only need to add the main `ntp` class; it contains the other classes from the module.

4. Click __Add class__.

5. Click __Commit 1 change__.

   **Note**: The `ntp` class now appears in the list of classes for the __NTP__ group, but it has not yet been configured on your nodes. For that to happen, you need to kick off a Puppet run.

7. From the command line of your Puppet master, run `puppet agent -t`.

8. From the command line of each PE-managed node, run `puppet agent -t`.

   This will configure the nodes using the newly-assigned classes.

> **Success!** Puppet Enterprise is now managing NTP on the nodes in the __NTP__ group. So, for example, if you forget to restart the NTP service on one of those nodes after running `ntpdate`, PE will automatically restart it on the next Puppet Enterprise run.

### Using the Events page to View Changes Made by the `ntp` Class

[EI-default]: ./images/quick/EI_default.png
[EI-class_change]: ./images/quick/EI_class-change.png
[EI-detail]: ./images/quick/EI_detail.png

The **Events** page lets you view and research changes and other events. For example, after applying the `ntp` class, check the **Events** page to confirm that changes were indeed made to your infrastructure.

Note that in the summary pane on the left, one event, a successful change, has been recorded for **Nodes: with events**. However, there are two changes for **Classes: with events** and **Resources: with events**. This is because the `ntp` class loaded from the `puppetlabs-ntp` module contains additional classes---a class that handles the configuration of NTP (`Ntp::Config`) and a class that handles the NTP service (`Ntp::Service`).

Click __With Changes__ in the __Classes: with events__ summary view. The main pane will show you that the `Ntp::Config` and `Ntp::Service` classes were successfully added when you ran PE after adding the main `ntp` class.

The further you drill down, the more detail you'll see. Eventually, you will end up at a run summary that shows you the details of the event. For example, you can see exactly which piece of Puppet code was responsible for generating the event. In this case, it was line 15 of the `service.pp` manifest and line 21 of the `config.pp` manifest from the `puppetlabs-ntp` module.

If there had been a problem applying this class, this information would tell you exactly which piece of code you need to fix. In this case, the **Events** page simply lets you confirm that PE is now managing NTP.

In the upper right corner of the detail pane is a link to a run report which contains information about the Puppet run that made the change, including logs and metrics about the run. See [Infrastructure Reports](./console_reports.html) for more information.

For more information about using the **Events** page, see [Navigating Events](./CM_events.html#navigating-events).


## Use the PE Console to Edit Parameters of the `ntp` Class

With Puppet Enterprise you can edit or add class parameters in the PE console without needing to edit the module code directly.

The NTP module, by default, uses public NTP servers. But what if your infrastructure runs an internal pool of NTP servers?

Changing the server parameter of the `ntp` class can be accomplished in a few steps using the PE console.

**To edit the server parameter of the** `ntp` **class**:

1. In the console, click __Nodes__ in the navigation bar.
2. On the __Classification__ page, select the __NTP__ group.
3. Click the __Classes__ tab, and find `ntp` in the list of classes.

4. From the __parameter__ drop-down list, choose __servers__.

   **Note**: The grey text that appears as values for some parameters is the default value, which can be either a literal value or a Puppet variable. You can restore this value by selecting __Discard changes__ after you have added the parameter.

5. In the __Value__ field, enter the new server name (for example, `["time.apple.com"]`). Note that this should be an array, in JSON format.
6. Click __Add parameter__.
7. Click __Commit 1 change__.
8. From the command line of your Puppet master, run `puppet agent -t`.
9. From the command line of each PE-managed node, run `puppet agent -t`.

   This will trigger a Puppet run to have Puppet Enterprise create the new configuration.

> Puppet Enterprise will now use the NTP server you've specified for that node.
>
> **Hint**: Remember to check the **Events** page to be sure the changes were correctly applied to your nodes!

## Other Resources

For more information about working with the puppetlabs-ntp module, check out our [Puppetlabs-NTP: A Puppet Enterprise Supported Module](http://puppetlabs.com/blog/puppetlabs-ntp-puppet-enterprise-supported-module) blog post and our [How to Manage NTP](http://puppetlabs.com/webinars/how-manage-ntp) webinar.

Puppet Labs offers many opportunities for learning and training, from formal certification courses to guided online lessons. We've noted a few below; head over to the [learning Puppet page](https://puppetlabs.com/learn) to discover more.

* [Learning Puppet](/learning/) is a series of exercises on various core topics about deploying and using PE.
* The Puppet Labs workshop contains a series of self-paced, online lessons that cover a variety of topics on Puppet basics. You can sign up at the [learning page](https://puppetlabs.com/learn).




