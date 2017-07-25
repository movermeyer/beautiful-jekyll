---
layout: page
title: Open Source
---

I love to contribute to Open Source projects. Besides being a maintainer of a [top 1000 Python library](https://github.com/closeio/ciso8601), my contributions have mostly been little bits here and there, fixing bugs as I find them in projects I use. You can find my contributions on the following platforms:

<div class="os-platform-list">
<ul class="list-inline text-center">
  <li class="list-inline-item">
    <a href="https://github.com/{{ site.social-network-links.github }}" title="GitHub">
      <span class="fa-stack fa-4x" aria-hidden="true">
        <i class="fas fa-circle fa-stack-2x"></i>
        <i class="fab fa-github fa-stack-1x fa-inverse"></i>
      </span>
      <span class="sr-only">GitHub</span>
   </a>
  </li>
  <li class="list-inline-item">
    <a href="https://gitlab.com/{{ site.social-network-links.gitlab }}" title="GitLab">
      <span class="fa-stack fa-4x" aria-hidden="true">
        <i class="fas fa-circle fa-stack-2x"></i>
        <i class="fab fa-gitlab fa-stack-1x fa-inverse"></i>
      </span>
      <span class="sr-only">GitLab</span>
    </a>
  </li>
  <li class="list-inline-item">
    <a href="https://bitbucket.org/{{ site.social-network-links.bitbucket }}" title="Bitbucket">
      <span class="fa-stack fa-4x" aria-hidden="true">
        <i class="fas fa-circle fa-stack-2x"></i>
        <i class="fab fa-bitbucket fa-stack-1x fa-inverse"></i>
      </span>
      <span class="sr-only">Bitbucket</span>
    </a>
  </li>
</ul>
</div>

# Specific Projects

## Foam

> [Foam](https://foambubble.github.io/foam/) is a personal knowledge management and sharing system inspired by Roam Research, built on Visual Studio Code and GitHub.

I took on the project of designing and implementing a unified user experience for creating notes from templates ("Templates v2").

This involved:

* Designing a new user experience for note templates:
  * Allowing for templates to be able to be used by all the note creation mechanisms, unifying their differing needs into a cohesive and simple mental model that users could reason about.
  * Extending the syntax of templates, in order to support useful metadata
* [Drafting the proposed changes](https://github.com/foambubble/foam/pull/534) as a proposal document; communicating clearly and asynchronously with all of the relevant stakeholders
* Actually implementing the changes. [More than a dozen PRs and counting...](https://github.com/foambubble/foam/pulls?q=is%3Apr+author%3Amovermeyer+)

This has been a solo passion project, and I'm very pleased with the results so far.

## Libraries Written

### ciso8601

[ciso8601](https://github.com/closeio/ciso8601) is the world's fastest Python library for parsing [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) timestamps.
I completely rewrote the existing ciso8601 library, creating version `2.0.0`.

As part of the rewrite I:

* Fixed the interface to have consistent error handling (the most asked for issue)
* Improved test coverage by automatically generating test cases for all valid formats.
* Made it >65% faster

It is now the fastest way to parse ISO 8601 in Python, by far!

### backports.datetime_fromisoformat

[backports.datetime_fromisoformat](https://pypi.org/project/backports-datetime-fromisoformat/) is a simple backport of Python 3.7's `datetime.fromisoformat` methods to earlier versions of Python 3.

### progress_tracker

[progress_tracker](https://github.com/exactEarth/ProgressTracker) is a simple (yet flexible) way to add processing progress logging to your Python scripts.

```python
>>> from progress_tracker import track_progress
>>> for _ in track_progress(list(range(1000)), every_n_records=100):
...     continue
...
100/1000 (10.0%) in 0:00:00.000114 (Time left: 0:00:00.001026)
200/1000 (20.0%) in 0:00:00.000274 (Time left: 0:00:00.001096)
300/1000 (30.0%) in 0:00:00.000374 (Time left: 0:00:00.000873)
400/1000 (40.0%) in 0:00:00.000473 (Time left: 0:00:00.000710)
500/1000 (50.0%) in 0:00:00.000572 (Time left: 0:00:00.000572)
600/1000 (60.0%) in 0:00:00.000671 (Time left: 0:00:00.000447)
700/1000 (70.0%) in 0:00:00.000770 (Time left: 0:00:00.000330)
800/1000 (80.0%) in 0:00:00.000868 (Time left: 0:00:00.000217)
900/1000 (90.0%) in 0:00:00.000979 (Time left: 0:00:00.000109)
1000 in 0:00:00.001086
```

## Bug Patches

* [Ruby on Rails](https://rubyonrails.org/): [#41381](https://github.com/rails/rails/pull/41381) (Minor)
* [Ansible](https://www.ansible.com/): [#24545](https://github.com/ansible/ansible/issues/24545) [#24546](https://github.com/ansible/ansible/issues/24546)
* [AtomLinter/linter-pyflakes](https://github.com/AtomLinter/linter-pyflakes): [(Minor)](https://github.com/AtomLinter/linter-pyflakes/commit/8c0a12478f138451c13fc32e0e3c5fe4116a947e)
* [retype](https://github.com/ambv/retype): [#1](https://github.com/ambv/retype/issues/1) [#2](https://github.com/ambv/retype/issues/2)
* [Construct](https://construct.readthedocs.io/en/latest/): [#360](https://github.com/construct/construct/issues/360) [#365](https://github.com/construct/construct/issues/365) [#372 (Documentation)](https://github.com/construct/construct/pull/372) [#398](https://github.com/construct/construct/issues/398)
* [pint](https://pint.readthedocs.io): [#387](https://github.com/hgrecco/pint/issues/387)
* [pyhocon](https://github.com/chimpler/pyhocon): [#53](https://github.com/chimpler/pyhocon/issues/53) [#132](https://github.com/chimpler/pyhocon/issues/132) [#134](https://github.com/chimpler/pyhocon/issues/134) [#135](https://github.com/chimpler/pyhocon/issues/135)
* [deepdiff](https://github.com/seperman/deepdiff): [(Documentation)](https://github.com/seperman/deepdiff/commit/10a117476c729a333b9b07f4d12c39adcb9c053c)
* [Cabot](https://github.com/arachnys/cabot): [Various](https://github.com/arachnys/cabot/issues?utf8=%E2%9C%93&q=author%3Amovermeyer%20)
* [Evolus Pencil](https://github.com/evolus/pencil): [#329](https://github.com/evolus/pencil/pull/329)
* [pyfirebirdsql](https://github.com/nakagami/pyfirebirdsql): [#61](https://github.com/nakagami/pyfirebirdsql/issues/61)
* [pyorbital](https://github.com/pytroll/pyorbital): [#6](https://github.com/pytroll/pyorbital/issues/6)
* [ansible-vault](https://github.com/jhaals/ansible-vault): [Various fixes](https://github.com/jhaals/ansible-vault/commits?author=movermeyer)
* [ansible-modules-hashivault](https://github.com/TerryHowe/ansible-modules-hashivault): [(Documentation)](https://github.com/TerryHowe/ansible-modules-hashivault/commit/a748049b2a5403507c3e84b24b44701ae5f9e9e5)
* [Blackberry2Droid](https://github.com/davehope/Blackberry2Droid): [#2](https://github.com/davehope/Blackberry2Droid/pull/2) [#3](https://github.com/davehope/Blackberry2Droid/pull/3)

## Bugs Reported

* [PostgreSQL](https://www.postgresql.org/): [#14289](https://www.postgresql.org/message-id/20160818174414.1529.37913%40wrigleys.postgresql.org)
* [GitLab](https://gitlab.com): [#31838](https://gitlab.com/gitlab-org/gitlab-ce/issues/31838)
* [autopep8](https://pypi.python.org/pypi/autopep8): [#333](https://github.com/hhatto/autopep8/issues/333)
* [NetworkX](https://networkx.github.io/): [#2134](https://github.com/networkx/networkx/issues/2134)
* [Evolus Pencil](https://github.com/evolus/pencil): [#331](https://github.com/evolus/pencil/pull/331) [#332](https://github.com/evolus/pencil/pull/332)
* [fastavro](https://github.com/tebeka/fastavro): [#49](https://github.com/tebeka/fastavro/issues/49) [#66](https://github.com/tebeka/fastavro/issues/66)
* [pyhocon](https://github.com/chimpler/pyhocon): [#102](https://github.com/chimpler/pyhocon/issues/102)
* [Esri/geometry-api-java](https://github.com/Esri/geometry-api-java/): [#75](https://github.com/Esri/geometry-api-java/issues/75)
* [Hypothesis](http://hypothesis.works/): [#379](https://github.com/HypothesisWorks/hypothesis-python/issues/379)
* [Cobbler](https://cobbler.github.io/): [#17220](https://github.com/cobbler/cobbler/issues/17220)
* [ciso8601](https://github.com/closeio/ciso8601/): [#26](https://github.com/closeio/ciso8601/issues/26)
* [pyfirebirdsql](https://github.com/nakagami/pyfirebirdsql): [#58](https://github.com/nakagami/pyfirebirdsql/issues/58) [#60](https://github.com/nakagami/pyfirebirdsql/issues/60)
* [akka](http://akka.io/): [#13962](https://github.com/akka/akka/issues/13962)
* [markdown-toc](https://github.com/AlanWalk/markdown-toc): [#50](https://github.com/AlanWalk/markdown-toc/issues/50)
* [PHYLO](https://github.com/McGill-CSB/PHYLO): [Various](https://github.com/McGill-CSB/PHYLO/issues?utf8=%E2%9C%93&q=is%3Aissue+author%3Amovermeyer+)
* Many, many, many more...
