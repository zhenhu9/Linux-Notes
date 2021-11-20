
# Redhat system registering

There is a No-Cost RHEL Developer Subscription available from 2016.

**No-cost Red Hat Enterprise Linux Individual Developer Subscription: FAQs**
https://developers.redhat.com/articles/faqs-no-cost-red-hat-enterprise-linux#general

**Redhat Ebooks**
https://developers.redhat.com/e-books

**Redhat Cheatsheets**
https://developers.redhat.com/cheat-sheets

## The relevant commands

subscription-manager, syspurpose

**subscription-manager**

Registers systems to a subscription management service and then attaches and manages subscriptions for software products.

subscription-manager command [options]

```
subscription-manager
	register
		--username=username
		--password=password
		--name=system_name
		--org=org		/* register your organization.

		--force			/* re-register your subscription.

		--activationkey=key	/* use key.
		--auto-attach	/* automatically attaches compatible subscriptions to this system.

	attach
		--auto		/* automatically attaches the compatible subscription.
		--pool=poolid	/* use subscriptions pool id to attach to the system.

	list
		--all		/* list all purchased subscriptions.
		--available	/* list available subscriptions which are not yet attached to the system.
		--consumed	/* list all of the subscriptions currently attached to the system.

	release
		--list		/* the available OS versions.
		--set=release	/* set the minor (Y-stream) release version to use.
		--unset		/* remove release version preference.
	
	refresh			/* refresh subscription data from the server.

	remove
		--pool=poolid	/* removes all subscriptions with the specifed pool id from the system.
		-all		/* remove all of the subscriptions from the system.

	unregister		/* remove the system's subscription and removes it from the subscription management service.

	/* equivalent to syspurpose command.
	role 		--set | --unset=VALUE
	service-level 	--set | --unset=VALUE
	usage 		--set | --unset=VALUE

	repos
		--list		/* list all repositories.
		--list-enabled	/* list all of the enabled repositories.
		--list-disabled /* list all of the disabled repositories.
		--enable=repo_id	/* enable the specified repo.
		--disable=repo_id	/* disabl the specified repo.

	facts
		--list		/* list the information about system and cpu.
		--update	/* update the information, if the machine hardwares are changed.

	identity		/* list identity information, include system name and organization name and id.

	status			/* show the current status of the products and attached subscriptions for the system.
```

**syspurpos**

Set the intended purpose for this system.

syspurpose command [option]

```
/* Set the intended purpose for this system.
syspurpose
	set-role VALUE		/* set the role property.
	unset-role VALUE	/* clear the role property.
		VALUE:
			Red Hat Enterprise Linux Server
			Red Hat Enterprise Linux Workstation
			Red Hat Enterprise Linux Computer Node
	
	set-sla VALUE		/* set the service level agreement property.
	unset-sla VALUE		/* clear the service lever agreement property.
		VALUE:
			Premium
			Standard
			Self-Support

	set-usage VALUE		/* set the system usage property.
	unset-usage VALUE	/* clear the system usage property.
		VALUE:
			Production
			Disaster Recovery
			Development/Test

	add-addons VALUE	/* add values to the add-ons property.
	remove-addons VALUE	/* remove values from the add-ons property.

	show			/* show the current of the system purpose property.

/* the other format of this command.
syspurpose
	set 	PROPERTY VALUE
	unset 	PROPERTY VALUE
	add 	PROPERTY VALUE
	remove 	PROPERTY VALUE	

```

Examples:

```
/* Registers the system. If there is only one subscrition,  --auto-attach will automatically attach it.
$ sudo subscription-manager register --username=username --password=password --auto-attach

/* If --auto-attach doesn't work, check out the pool id of your subscription and attach it,
$ sudo subscription-manager attach --pool=pool_id

/* Then set the role, usage and service level properties.
$ sudo subscription-manager role --set="Red Hat Enterprise Linux Server"
$ sudo subscription-manager usage --set="Development/Test"
$ sudo subscription-manager service-level --set="Self-Support"

/* Then check out the OS release version that you can choose.
$ sudo subscription-manager release --list

/* Then set the release version which you choose.
$ sudo subscription-manager release --set=8.2

**Notes**: If it doesn't work well when attaching the system to the subscription, you can refresh the subscription data from the server by using `subscription-manager refresh`.

/* Now if you update the system, the release version will be fixed with 8.2.
$ sudo dnf update
```
