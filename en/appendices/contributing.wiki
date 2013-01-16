# Contributing to Lithium

Thank you for your interest in contributing to Lithium! This project is built by a thriving community of developers who value cutting-edge technology and concise, maintainable code. If you've found a bug, or have an idea for a feature, we encourage your participation in making Lithium better.

## tl;dr

 _In a hurry? Here's what you need to stick to in order to have the best chance of getting your code pushed to the core:_
<br /><br />

 * **Conceptual integrity**: code should conform to the goals of the framework
 * **Maintainability**: code should pass existing tests, have adequate test coverage and should conform to our coding standards & QA guidelines
 * **Comprehensibility**: Code should be concise and expressive, and should be accompanied by new documentation as appropriate, or updates to existing docs
 * **Integration**: Finally, pull requests should be submitted against the [`dev`](https://github.com/UnionOfRAD/lithium/tree/dev) branch for integration testing

## Types of Contributions

Whatever you'd like to contribute to the project, you may wish to discuss it with the core team and development community before beginning your work. This can happen in one of two ways: you can [open an appropriately-labeled issue](https://github.com/UnionOfRAD/lithium/issues/new) in GitHub, or you can chat directly with a core team member by joining the [#li3-core](irc://irc.freenode.net/#li3-core) IRC channel on Freenode.

We value collaboration and believe that the best solutions are usually found through use-case-oriented discussions and&mdash;occasionally&mdash;some healthy debate. If you have an idea for a killer feature or think you've found a bug, please talk it over with a seasoned member of the team on IRC.

Combining great new ideas with the wisdom of experience will help us create the best possible features for Lithium.  Likewise, talking through a bug with a core team member will help us ensure the best possible fix.  We're striving to maintain a lean, clean core and want to avoid introducing patches for symptoms of an underlying flaw.

### Enhancements and New Features

One of Lithium's key goals is a light, clean core, which means careful consideration of new features. Here are some of the criteria we go by when deciding whether or not to incorporate a new feature into the framework:<br /><br />

 * **Does it fit within the existing set of features?** Lots of features are great ideas in their own right, but might not be right for integration with the core itself. Examples include wholly new pieces of functionality that could easily fit within a plugin, or adapters for technologies that aren't widely used.<br /><br />Often, even in cases where integration for a widely-used technology might otherwise make an obvious addition, we choose to keep things in plugins in order to ensure the core always stays as light as possible. Every new feature introduced has a permanent cost in terms of maintenance and documentation, increases testing burden and couples additional code to every release cycle.
 
 * **Is the feature not easily replicated with a few lines of app code?** Sometimes it's tempting to implement a new option flag or class property that makes your application code a little more convenient in a small subset of cases, but a lack of careful editing leads not only to high maintenance overhead, but to a lack of long-term flexibility in evolving the API. The framework is designed to be modular and extensible, and should be treated as such.<br /><br />If there's enough code to justify a plugin, publish it as such. Many times, features that don't make the cut in one version are implemented as plugins which gain wide acceptance, and are integrated into the core in subsequent versions.
 
If your idea passes these two basic tests, it's time to engage a core team member to get started on the idea. Every new feature is handled differently, but the basic workflow usually ends up playing out something like this:

 1. Contacting a core team member via email, or GitHub issue ticket
 2. Use-case oriented discussions (complete with healthy debate!)
 3. Initial implementation, often in a pre-identified branch or fork
 4. Test case and documentation development (or updates)
 5. Code review and merge into mainline
 
### Bugs

Helping to fix bugs is a huge help to the core team, who is often burdened with more they can already handle. Fixing exising issues is also a great way to free up core team time for adding new feaures!

Helping with bugs follows the same 5-step process outlined above: just be sure to check in with someone on the core team to understand how to best help. This will make sure that you're workign on bugs that have a high-priority and aren't being investigated by anyone else.

If you're looking to report an issue, there are a few things you'll want to consider before submitting an issue on one of our GitHub repos:

 * Make sure you're using the latest code. It's possible your issue is already fixed! 
 * Search the issues list to make sure the problem hasn't already been reported.
 * Add a descriptive title and summary. Phrases like "doesn't work" aren't really helpful without lots of supporting details.
 * Submit a test case with the issue. This helps us both reproduce the issue and also verify possible fixes.
 
Above all, remember that _volunteers_ are going to be responding to these requests for help. Politeness and respect go a long ways, as do humor, preventative research, and bribery.

### Security Vulnerabilities

Security vulnerabilities are an especially sensitive class of bug and should be confidentially reported directly to [security@union-of-rad.org](mailto:security@union-of-rad.org). Please do not disclose any details publicly. When reporting security vulnerabilities, please specify the version affected, include relevant reproduction code along with any other pertinent information relevant to addressing the vulnerability, such as 3rd-party software or components, etc.

On reporting a confirmed security vulnerability, you can expect to receive a response from a core team member within 24 hours containing next steps as well as any follow-up questions necessary to produce a patch and publish a security update.

### Documentation

Lithium documentation is an ongoing work, and we need your help. Lithium users _new_ and _old_ are welcome and encouraged to join in the fun. If you've struggled with something, help us record and share the solution so others can find the way more easily. There are many ways you can help:

 * Code examples to enrich current documentation
 * Creation of lists or tables to help explain concepts
 * Rough notes that describe an oft-used process
 * Corrections (typos, inaccuracies, etc.)
 * Translations

If you'd like to help, simply [fork the project on GitHub](https://github.com/UnionOfRAD/manual), [open a new issue](https://github.com/UnionOfRAD/manual/issues), or get in contact with one of the core team members. You can give us a shout in channel #li3 on server Freenode, or send an email to anderson.johnd at gmail dot com.


## Branching

The Lithium core is managed on a very simple branching workflow: when developing new features or bug fixes, a topic branch with a relevant name is created, such as `new-media-encode` or `model-find-fix`. Using comprehensible branch names helps us make sense of the source history as branches are merged.

Once commits on a topic branch have been verified through testing, are properly documented and pass our QA checks, the topic branch will be merged into the `dev` integration branch. Ordinarily, you'll want to point your pull requests at this branch.

For long-running feature branches, like `data`, you can point relevant pull requests there instead.

<table style="width: 50%; border: 0; margin-top:20px">
	<tr>
		<td style="border: 0;">Master Branch (Stable)</td>
		<td style="border: 0;">
			<a href="http://travis-ci.org/UnionOfRAD/lithium" style="border: 0; padding: 0;">
				<img
					src="https://secure.travis-ci.org/UnionOfRAD/lithium.png?branch=master"
					alt="Build Status: Master"
				/>
			</a>
		</td>
	</tr>
	<tr>
		<td style="border: 0;">Development / Integration Branch (Unstable)</td>
		<td style="border: 0;">
			<a href="http://travis-ci.org/UnionOfRAD/lithium" style="border: 0; padding: 0;">
				<img
					src="https://secure.travis-ci.org/UnionOfRAD/lithium.png?branch=dev"
					alt="Build Status: Dev"
				/>
			</a>
		</td>
	</tr>
</table>

