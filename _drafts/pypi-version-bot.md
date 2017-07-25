---
layout: post
title:  "PyPI version tags bot"
date:   2018-01-09 13:30:01 -0000
categories: python
---

TODO: Update post time.

**TL;DR:** I wrote a bot that checked PyPI and GitHub for Python packages that didn't advertise their supported Python versions and creates issues and Pull Requests to fix this.

Back in December, I was porting a codebase from Python 2 to Python 3. Part of that task is making sure that all of the required dependencies also have support for Python 3. While nearly all of the packages you're likely to come accross have had Python 3 for years, this codebase had some obscure dependencies. I jumped over to PyPI and found that they had neither Trove classifiers, nor did they have any mention of supported versions in their README files on GitHub. *sigh*

Taking a look at the codebase, it looked as if both had Python 3 support. But it shouldn't require code inspection to do this. I opened issues on both of the repos asking for the supported versions to be added to the classifiers. Both issues were enthusiastically received and quickly addressed. That felt good, which got me wondering if I good do this automatically.

But how would something like that work? In my mind, it would go something like:
    1. Get the list of packages from PyPI
    1. Fetch the Trove version classifiers for each package
    1. For those that are missing them, fetch the README and check for declared versions there
    1. Create an issue and possibly even a Pull Request that makes it simple for maintainers to simply click "Merge"

This way, the Python community (myself included) would benefit from more packages having proper version tags (And I could pad my GitHub commit activity, [which is apparently useful if you are looking for a job.](https://medium.com/@WebReflection/the-day-my-raspberry-pi-failed-at-faking-my-github-activity-5ed65d73dd06)).

The rest of this post will discuss the how I went about building this and the surprise challenges that had to be overcome. If you just want to see the end result, jump down to the conclusion.

# Getting the list of PYPI packages

At time or writing, there are 126,243 packages on PyPI. That seemed like a lot, and I suspected that a large percentage of the are abandoned or unmaintained. So I'd really like to be able to get the list of packages in descending order of popularity.

Thankfully, the [dedicated folks at PyPI](https://caremad.io/posts/2016/05/powering-pypi/) [publish a record for every download request](https://mail.python.org/pipermail/distutils-sig/2016-May/028986.html) in a [BigQuery](https://cloud.google.com/bigquery/) dataset. 

Anyone with a Google Account and a credit card can spend a few cents and query the dataset using Google Cloud Platform.

I had never used GCP before. Luckily, Google offers 12-month free trial, with $300 USD of free credit to start you off. Getting set up was trivial, and I quickly run a query that would get me the total # of downloads for every PyPI package in the past 6 months (arbitrary length of time):

```
SELECT
  file.project,
  COUNT(*) as total_downloads,
FROM
  TABLE_DATE_RANGE(
    [the-psf:pypi.downloads],
    TIMESTAMP("20170601"),
    CURRENT_TIMESTAMP()
  )
GROUP BY
  file.project
ORDER BY
  total_downloads DESC
``

It took roughly 20 seconds to run and cost roughly $0.20 CAD. Isn't the cloud wonderful? 😊
(Actually my entire experience with GCP was excellent. Kudos to the fine folks at Google for making everything "just work" for beginners)

The results were interesting. There were 5,302,388,010 package downloads in the time period. Looking at the top 20, the results are probably familiar to most Python developers (Aside: if you are a Python developer and haven't heard of any of these, check them out. They might just make your life much easier):

| #  | Package         | # of Downloads |
|----|-----------------|----------------|
| 1  | simplejson      | 147,501,833    |
| 2  | six             | 121,053,361    |
| 3  | botocore        | 103,975,534    |
| 4  | python-dateutil | 101,406,597    |
| 5  | setuptools      | 98,674,463     |
| 6  | pip             | 97,394,124     |
| 7  | requests        | 90,541,110     |
| 8  | pyyaml          | 90,440,557     |
| 9  | pyasn1          | 89,285,720     |
| 10 | s3transfer      | 88,432,197     |
| 11 | futures         | 87,603,204     |
| 12 | docutils        | 86,009,260     |
| 13 | jmespath        | 80,371,942     |
| 14 | awscli          | 74,338,936     |
| 15 | rsa             | 70,448,395     |
| 16 | colorama        | 67,716,151     |
| 17 | idna            | 59,897,753     |
| 18 | certifi         | 56,718,258     |
| 19 | wheel           | 54,645,839     |
| 20 | urllib3         | 54,440,433     |

The ext interesting thing to note was that only 118,210 packages were in the result set. This means that 6.5% of packages on PyPI haven't been downloaded in the past 6 months (35% have had less than 1000 downloads). The distribution of downloads looks like:

<log scale distribution>

Note that this is a log (base 10) scale. Predictably, its top-heavy with the 543 most popular packages capturing 80% of the downloads, 13,682 has 95%, 59.978 have 99%. While you could run the query yourselves, I've dumped the list for your own perusal [here]().

Now that I had the list of packages, I now needed to find out which of them were missing Python version Trove classifiers. After a bit of poking around, I discovered that you could simply add "/json" to the PyPI page and get back an easily parseable version, including the classifiers and homepage URL. So I wrote a simple script to pull down this information and dump it into a [SQLite](https://en.wikipedia.org/wiki/SQLite) DB for each package.

The only complexity here was the fact that some of the package URLs resulted in 404s (TODO: look into why). So in those cases, I added them to a blacklist so I wouldn't try to process them again. There ended up being were 2,411 of these.

At this point, it was time for a sanity check. Before going further, we should confirm some hypotheses:

1. Many packages are missing the supported version classifiers
1. Most packages are hosted on GitHub

A quick query of the database showed that there were 60,774 packages (TODO: percent) on PyPI without version classifiers. 3,132 of the top 10,000 packages were missing them.

# Where are Python packages developed?

We had downloaded the `home_page` attribute of each package from PyPI. But as with any human-populated free-form text field, there were a bunch of data quality issues.

Some package maintainers use this field to print to the git repo or master branch URL directly. Some projects have their own websites and use the field to contain that URL. Some are out of date and point to old project names. Many are populated with garbage like `"UNKNOWN"`, `"null"` and `"tbd"`.

After removing the obviously bad entries, here are the top 10 domains mentioned:

| #  | Domain            | # of Packages with domain in `home_url` |
|----|-------------------|-----------------------------------------|
| 1  | github.com        | 75119                                   |
| 2  | bitbucket.org     | 3819                                    |
| 3  | python.org        | 3398                                    |
| 4  | plone.org         | 956                                     |
| 5  | github.io         | 923                                     |
| 6  | google.com        | 805                                     |
| 7  | headfirstlabs.com | 760                                     |
| 8  | readthedocs.org   | 584                                     |
| 9  | gitlab.com        | 547                                     |
| 10 | sourceforge.net   | 422                                     |

No one has captured the hearts and minds of developers quite like GitHub. Its importance is still clear once you sort by total downloads:

| #  | Domain              | # of Packages with domain in `home_url` | Total # of downloads for packages hosted on domain |
|----|---------------------|-----------------------------------------|----------------------------------------------------|
| 1  | github.com          | 75119                                   | 2,933,783,950                                      |
| 2  | readthedocs.io      | 378                                     | 240,859,348                                        |
| 3  | python.org          | 3398                                    | 198,391,668                                        |
| 4  | pypa.io             | 7                                       | 129,073,177                                        |
| 5  | bitbucket.org       | 3819                                    | 124,361,978                                        |
| 6  | openstack.org       | 322                                     | 119,465,493                                        |
| 7  | amazon.com          | 6                                       | 119,134,300                                        |
| 8  | readthedocs.org     | 584                                     | 113,932,892                                        |
| 9  | sourceforge.net     | 422                                     | 92,411,499                                         |
| 10 | python-requests.org | 5                                       | 91,232,460                                         |

The results so far were promising. 76,128 packages had the word "github" in their url domains, and 35,253 of those were missing version classifiers.

# Getting the GitHub project URLs

Since I was limiting my scope to projects that were hosted on GitHub, I started by looking for any url that had "github" in the domain.

```
https://github.com/PipelineAI/pipeline
https://github.com/ipython-contrib/jupyter_contrib_nbextensions.git
http://ryan-roemer.github.io/sphinx-bootstrap-theme/README.html
https://github.com/maxeventbrite/zendesk/tree/master"
http://numba.github.com"
https://www.github.io/xiyouMc/"
```

It turns out that there are a lot of ways to reference a GitHub repo. I would like to normalize the representation. I created a trivial no-op sanitization function:

```
def sanitization(url, package):
    return url
```

I would then try fetching the URLs after running them through the sanitization function, look at the list of failures and tweak the sanitization function to correct them.

So the above list became:

```
PipelineAI/pipeline
ipython-contrib/jupyter_contrib_nbextensions
ryan-roemer/sphinx-bootstrap-theme
eventbrite/zendesk # (hand-entered correction)
numba/numba
xiyouMc/vivaredis # (in cases like this, append the package name)
```

Now that I had a function that sanitizes and normalizes the URLs, it was a simple matter to create a loop that filled in a `project` column in the same SQLite database.

Runnning a simple GET request for each project page returned 400 (TODO: real number) HTTP 404 status codes, indicating that some projects have been removed from GitHub. 

## Read The Docs
I also realized that many of the projects with URLs "readthedocs" in their domain name would also have their source code hosted on GitHub. So I wrote another function that would attept to determine the GitHub URL by parsing the Read the Docs URL and applying some heuristics:

  1. Fetch the Read The Docs URL
  1. Find all links on the page that point to github.com
  1. Filter the links to those that contain the package name
  1. If there is only one remaining link, use that as the GitHub project page.

This got me an additional 472 (TODO: Get real number) GitHub project locations (for a total of 76,588 locations), without much additional effort.

## Progress so far

At this point, I knew the github project URL and the fact that they were missing the classifier tags. I technically have enough that I could create an issue on each repo. 
But what would I say?  At this point, all I could say is "Hi, I noticed you're missing version classifiers in your setup.py. Could you please fix this?".

That wasn't satisfying for me, nor as a library maintainer would I want to receive such a ticket. 

We should at least make an attempt of doing better detection of the versions they support, and make it easy for maintainers to make the changes. Perhaps we could even get enough confidence to make them a pull request.

The first hint we could use to detect their supported versions is to search their READMEs. 

# Fetching READMEs

We had the project URLs. At first I tried just appending "README.md" to each project URL:

```python
GITHUB_PATH = "https://raw.githubusercontent.com/{project}/master/README.md"
for project in projects:
    response = requests.get(GITHUB_PATH.format(project=project))
```

But of course, not all projects host their READMEs at `README.md`. For starters, many use other extensions (`.markdown`, `.rst`, `.mkd`, `.txt`). Further, not every project has a `master` branch (ex. [python-phonenumbers](https://github.com/daviddrysdale/python-phonenumbers) uses `dev`). Instead of continuing down this path of trying all known combinations of README locations, I had to change tactics.

## BEA-Utiful Soup to the rescue

When you visit a GitHub repo, it renders the README file. More importantly, it renders the name of the README file:

<Screenshot of rendered name>

Using the wonderful [Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/), we could easily parse out the README path:

TODO: Add code?

# Parsing READMEs

READMEs are by their very nature, free-form human language text.

In order to find a list of supported Python versions in a README file TODO: Complete

## README Language Detection

While I was parsing the README's, I also ran the text through the [`guess-language-spirit`](https://bitbucket.org/spirit/guess_language) library, which attempts to guess what the human language in the README is. Knowing this would be useful if I wanted to produce issues in the same language as the README:

Top 5 Languages in READMEs (according to `guess-language-spirit`) (TODO)

| #  | Language | # of Packages with READMEs in that language |
|----|----------|---------------------------------------------|
| 1  | en       | 65125                                       |
| 2  | ca       | 940                                         |
| 3  | UNKNOWN  | 916                                         |
| 4  | fr       | 323                                         |
| 5  | pt       | 230                                         |

Note that `UNKNOWN` means that `guess-language` wasn't able to make a good guess for the language. For example, it is impossible to determine the language of an empty README. 

`ca` is Catalan, which seemed surprising. Manual inspection of those README's suggests that for whatever reason, markdown is being classified as Catalan. The `fr` (French) and `pt` (Portuguese) seem to be more accurate, but both of these also contain lots of English README's as well.

If we want a better guess, we'll have to go elsewhere. I tried using [Google's language detection API](https://cloud.google.com/translate/docs/detecting-language), but [their costs](https://cloud.google.com/translate/pricing?csw=1) were far too restrictive to apply to all of the READMEs. As a compromise I limited the requests to those that `guess-language-spirit` determined were not English. That way, I'd get confirmations of the ones I'm least confident in. Here is the same list using Google's recognition:

Top 5 Languages in READMEs (using Google's language detection for non-English `guess-language-spirit` results) (TODO)

| #  | Language | # of Packages with READMEs in that language |
|----|----------|------------------------------------------------------------------|
| 1  | en       | 67777 (65125 from `guess-language-spirit`)  |
| 2  | zh-CN    | 268                                         |
| 3  | pt       | 161                                         |
| 4  | es       | 134                                         |
| 5  | UNKNOWN  | 132                                         |

All this just shows that English dominates the languages found in READMEs. So much so that any effort spent on translation/[i18n](http://www.i18nguy.com/origini18n.html) efforts would not yield many gains. (TODO: Do analysis by download count)

# Release Wheels

The next thing we could do is check the releases page to see if they are building Python 3 specific wheels. Then, we could also look at the source files themselves to look for telltale signs of specific Python versions.

# Parsing the code

Finally, the last clue I used to figure out the supported versions of Python was to determine which versions of Python they did not support. 
There have been many syntax changes throughout the versions of Python. If we noticed that a project's source code was making use of specific syntax features, then we could rule out the versions of Python that are missing support for those features.

This would require statically analyzing the source code. I wasn't going to actually run the code: YOU SHOULD NEVER RUN UNTRUSTED CODE. It could do any sort of nasty things to your computer, and you have no idea what malicious code might be sitting around in random GitHub repos.

But even thought I was only going to statically analyze the code, there is still risk. I would be using git to clone the repo, which would download the source code to my computer. If someone had discovered a way to exploit a vulnerability in the `git` executable, there would be a possibility that they could compromise my computer. Further, if there were vulnerabilities in any of the static analysis code I was using, I could also find myself in trouble. 

Thankfully, I had thought about these sorts of attacks and I was already running within a virtual machine (In a similar fashion, if someone had figured out a way to exploit Beautiful Soup 4 my computer could have been compromised). This is by no means perfect protection, but it does make an attacker's job significantly more difficult ([which is the entire goal of security]() TODO: Find link about the economics of security).

I started by using the [`GitPython`](https://github.com/gitpython-developers/GitPython) package to clone each repository. For each Python source file (`.py`) I would try an guess which versions of Python were excluded by the use of unsupported syntax features.

At first, I wrote some heuristics based on syntax changes that I knew. But that very quickly became tiresome. Then I remember the `ast` module. `ast` is a built-in module that tokenizes and parses Python source code in exactly the same that the Python interpreter does. So I could simply try to parse each source file with `ast` and if the parse failed, I would know that the file is not parseable with that version of Python.

```
TODO: Code
```

The only snag here was that `ast` is tied to the version of the interpreter you are using, and cannot parse source code with previous versions of the syntax. So my Python 3.6 `ast` module could not parse the source file with the Python 2.7 syntax.

I made a quick hack to get around this problem: I installed every version of the Python interpreter (`ast.parse()` was introduced in Python 2.6, so I installed only versions newer than that). Then all I had to do was shell out to each version in turn and see if their version of the `ast` parse failed or not:

```
TODO: Code
```

This worked fairly well. TODO: Write more about results.

However, it was slow. In order to speed it up, I implemented a handful of heuristics and shortcuts. Firstly, I ignored empty `.py` files (ex. many `__init__.py` files) by checking the file size before attempting the parse. I also found the [`yapf`](https://github.com/google/yapf) and its `_GRAMMAR_FOR_PY2` and `_GRAMMAR_FOR_PY3` attributes, which can be used to determine the broad versions supported (Python 2, Python 3, or both):

```
from yapf.yapflib.pytree_utils import _GRAMMAR_FOR_PY2, _GRAMMAR_FOR_PY3
try:
    parser_driver = driver.Driver(_GRAMMAR_FOR_PY2, convert=pytree.convert)
    parser_driver.parse_string(source + '\n', debug=False)
except parse.ParseError as exc:
    print("This code doesn't support Python 2")
```

With this quick check I could often eliminate checking the interpreters that belonged to either Python 2 or Python 3.


# Creating Issues 

With all this information, I could then produce GitHub issues that I hoped would be appreciated instead of being considered a nuisance. 

In the case where I was quite confident that I had determined their supported versions, I also had the script produce Pull Requests that maintainers can accept immediately with a click of the merge button. 

## Badges

Badges are those cute little colourful rectangles that you see at the top of many READMEs. In the case where the project was already using badges, the script also added the "supported Python versions" badge as part of the pull request.

Further, some projects made use of the shuttered [pypip.in badge service](https://web.archive.org/web/20150318013508/https://pypip.in/). I ended up making a second bot that made pull requests for projects that were using those URLs, switching them to use the [shields.io](https://shields.io) version.

TODO: Self Check-out Page
TODO: Rate Limiting of Release
TODO: Response Rate + Results

# Issue Tracking

It is one thing to automatically open issues on thousands of GitHub repos. It would be beyond the pale to do so without having a mechanism to track them.

I created a new database table. Each time an issue was opened in a project's issue tracker, I logged an entry in this table.
I then created a script that would visit each of the issues and determine the state of the issue (open, closed with pull request, closed without pull request, closed with referenced commit), number of comments on the issue, time of last change, and assigned user.

I also made a tool that would allow me to easily manually change the state.

# Types of Issues and Pull Requests

In the process of doing all this work, I realized that there were several cases where I could provide a valuable issue or pull request:

- In cases where the version classifiers were missing, I can try to detect the supported versions and create an issue.
- In cases where the source code doesn't have a consistent supported python version, I can let them know that perhaps they missed a file in their porting.
- In cases where the version classifiers don't match the source or README, I can let them know.
- In cases where they are already using badges in their readme (and have version classifiers), I can create a pull request to add the version one.

# Curious case of `russianidiot`
TODO: