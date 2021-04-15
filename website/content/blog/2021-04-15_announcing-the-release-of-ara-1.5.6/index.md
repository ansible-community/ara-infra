---
author: "David Moreau Simard"
categories:
  - news
tags:
  - ansible
date: 2021-04-15
title: "Announcing the release of ARA 1.5.6"
slug: announcing-the-release-of-ara-1.5.6
type: post
---

ARA 1.5.6 has been released and you can try it out with the [getting started guide](https://ara.readthedocs.io/en/latest/getting-started.html)
or by checking out the live demo at https://demo.recordsansible.org.

For the full list of changes, see the [changelog on GitHub](https://github.com/ansible-community/ara/releases/tag/1.5.6)
as well as the list of [commits since 1.5.5](https://github.com/ansible-community/ara/compare/1.5.5...1.5.6).

It's been a while since the last [release highlight for 1.5.0](https://ara.recordsansible.org/blog/2020/09/23/announcing-the-release-of-ara-1.5.0/)
but you can catch up on changelogs and release notes for every version until now in the [documentation](https://ara.readthedocs.io/en/latest/changelog-release-notes.html).

1.5.6 features a refresh of the playbook reporting interface included with the API server and here you can find some of
the highlights.

## A refreshed UI

We've switched to the [bootstrap CSS framework](https://getbootstrap.com/) which made it possible to theme the interface
with [bootswatch](https://bootswatch.com/):

{{< tweet 1371294874864119811 >}}

There are some fun themes (like [sketchy](https://bootswatch.com/sketchy/)) but we ended up settling for
[flatly](https://bootswatch.com/flatly/) for the default light theme and [darkly](https://bootswatch.com/darkly/) as
the dark theme.

The first implementation for themes was configured in the server's settings but it wasn't great because users could not
change the theme from their browser. Thanks to the help of [Guillaume Vincent](https://twitter.com/guillaume20100) we
ended up implementing a toggle in the interface which persists across page loads and it works great:

{{< tweet 1381385908612698119 >}}

It turns out that the themes have a shade of green which gives the interface a bit more personality and is closer to the ara logo.

I prefer the dark theme myself but some have already told me they preferred the light one.

To each their own and that's alright :)

## A different mobile interface

We didn't particularly spend a lot of time on the mobile use case for now but the interface at least needed to be usable.

It now behaves differently than it used to and I personally like it better this way:

{{< tweet 1370985429386862592 >}}

## Iterations, iterations, iterations...

The data that is available in the interface is the same as before but the way it's presented has been generally improved by iterating, iterating and iterating:

{{< tweet 1374745820214484999 >}}

And then some more iterations after that -- tweaking colors, heights, alignments, modals and so on -- got us to a point where it was "good enough" to ship.

*Special thanks to [@hille721](https://twitter.com/hille721) on GitHub who contributed a lot of feedback and ideas throughout [issues](https://github.com/ansible-community/ara/issues/229) and [pull requests](https://github.com/ansible-community/ara/pull/221) while we were iterating !*

# That's it for now !

There's plenty of work left to do but it will need to be in a future release !

## Want to contribute, chat or need help ?

ARA could use your help and we can also help you get started.
Please reach out !

The project community hangs out on [IRC and Slack](https://ara.recordsansible.org/community/).

You can also stay up to date on the latest news and development by following [@RecordsAnsible](https://twitter.com/RecordsAnsible) on Twitter.

