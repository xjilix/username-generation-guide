# OSINT Username generation guide

A definitive guide to generating usernames for OSINT purposes.

![Logo](./pictures/logo.png)

## Start

Let's find out your goals.

If I understand correctly, **you have some information about people**, and you want to **get a list of usernames** (nicknames), that's may be used to search for those people.

Am I right? So, you're in right place.

Below you can read the information on how to get clues for a new search, starting from the data you know, as well as how to automate this and what tools to use.

### What do you have?

If you only have some information as a first name, a last name, a birthday (and, maybe some extra info), you should take a look at the section [“Combining primary info"](#combining-primary-info).

Do you need extra help to extend the number of likely usernames? For learning methods to get variants of first names and so on, check section [“Primary info mining”](#primary-info-mining).

If you have a username and want to guess similar usernames, jump to the [“Username transformations”](#username-transformations) section.

**Important!** Clone this repository with `git` or [download it](https://github.com/soxoj/username-generation-guide/archive/refs/heads/main.zip) to use Python scripts mentioned below.

## Combining primary info

Usernames/logins commonly consist of a combination of a first name, a last name, and, a little less often, a middle name (patronymic). Only the first letters can be left, and parts can be separated by some characters as `_`, `.` and so on.

Of course, there can be many such combinations, so automation tools are needed. A good example is very useful interactive [Google spreadsheet](https://docs.google.com/spreadsheets/d/17URMtNmXfEZEW9oUL_taLpGaqTDcMkA79J8TRw4xnz8/edit#gid=0) for email permutations from Rob Ousbey, from Distilled.net.

Here is an example of use for `rob ousbey`:

```sh
rob@distilled.net
ousbey@distilled.net
robousbey@distilled.net
rob.ousbey@distilled.net
rousbey@distilled.net
...
```

Also, you can find it convenient to use [Email Permutator](http://metricsparrow.com/toolkit/email-permutator/) from Metric Sparrow Toolkit.

For fans of a console there are some specialized tools:

- Script [python-email-permutator](https://github.com/Satys/python-email-permutator) based on spreadsheet mentioned above.

- [Logins generator](https://github.com/c0rv4x/logins-generator) supporting flexible ways to combine first, last and middle names.

- [emailGuesser](https://github.com/WhiteHatInspector/emailGuesser) well customizable permutator with support of checks if address is valid in Skype and in breach databases. 

Looking ahead, I will tell you that from lists of names you can [quickly make](#addition-of-mail-domain) a list of emails.

---

If you have any other additional information, you can significantly expand the number of candidates for usernames. It can be a year of birth, city, country, profession, and... literally anything.

What can be used in this case?

- My own script based on ProtOSINT combination methods:

```sh
$ python3 generate_by_real_info.py
First name: john
Last name: smith
Year of birth: 1980
Username (optional):
Zip code (optional):
johnsmith1980
smith
johnsmith80
jsmith1980
smithjohn
...
```

- Great alias generator mode of [OSRFramebork](https://github.com/i3visio/osrframework):

```sh
$ osrf alias_generator
Insert a name:                     john
Insert the first surname:          smith
Insert the second surname:
Insert a year (e. g.: birthyear):  1980
Insert a city:
Insert a country:

Additional transformations to be added
--------------------------------------

Extra words to add (',' separated):

Input data:
-----------

Name:               john
First Surname:      smith
Year:               1980

Generated nicks:

[
  "j.smith",
  "j.smith.1980",
  "j.smith.80",
  "j_smith",
...
Up to 41 nicks generated.

Writing the results onto the file:
	./output.txt
```

[↑ Back to the start](#what-do-you-have)

## Primary info mining

It is can be very important to check all the variants of non-English usernames. For example, a person with the common name *Aleksandr* may have a passport with the name `Alexandr` (letter `x`) and a working login starting with `alexsandr` (letters `xs`) because of the different transliteration rules.

This is a source of variability for us, so let's use it.

- [BehindTheName](https://www.behindthename.com/name/john) - excellent site about names. There are common name variants, diminutives (very useful for personal logins), and other languages alternatives.

!['Aleksandr' name variants](./pictures/behindthename.png)

You can use a simple script to scrape such data:
```sh
$ python3 behind_the_names.py john diminutives
Johnie
Johnnie
Johnny
```

- [WeRelate](https://www.werelate.org/wiki/Special:Names) - Variant names project, a comprehensive database of name variants with the ability to search. Gives much more results than BehindTheNames, but there are also many irrelevant results. Also, see [GitHub repo](https://github.com/tfmorris/Names) with project data.

[↑ Back to the start](#what-do-you-have)

## Username transformations

When you sign up on the site it may turn out that your username is taken. Then you use a variant of a name - with characters replacement or additions.

Thus, making assumptions about the transformations and knowing the original name, you can check "neighboring" accounts (for example, with [maigret](https://github.com/soxoj/maigret)).

I propose for this my own simple tool that allows you to make transformations by rules.

```sh
$ python3 transform_username.py --username soxoj rules/printable-leetspeak.rule
soxoj
s0xoj
5ox0j
50xoj
...
```

Rules for transformation are located in the directory `rules` and consist of the following:

- `printable-leetspeak.rule` - common leetspeak transformations such as `e => 3`, `a => 4`, etc.
- `printable-leetspeak-two-ways.rule` - the same conversions from letters to numbers, but also vice versa
- `impersonation.rule` - common mutations used by scammers-impersonators such as `l => I`, `O => 0`, etc.
- `additions.rule` - common additions to the username: underscores and numbers
- `toggle-letter-case.rule` - changing case of letters, what is needed not so often, but maybe useful
- `add_email.rule` - custom rule to add mail domain after usernames

You can use a file with a list of usernames:

```sh
$ cat usernames.txt
john
jack

$ python3 transform_username.py rules/impersonation.rule --username-list soxoj
jack
iack
john
iohn
```

And even use a pipe to use the output of other tools and itself, combining transformations:
```sh
$ python3 transform_username.py rules/printable-leetspeak.rule --username soxoj | python3 transform_username.py rules/impersonation.rule  -I
s0xOj
sOx0j
5OxOi
soxOj
sox0i
...
```

#### Addition of mail domain

You can use `add_email.rule` and easily edit it to add needed mail domains to check emails in tools such as [mailcat](https://github.com/sharsil/mailcat), [holehe](https://github.com/megadose/holehe), or [GHunt](https://github.com/mxrch/GHunt).

```sh
$ python3 transform_username.py rules/printable-leetspeak.rule --username soxoj | python3 transform_username.py rules/add_email.rule --remove-known -I
soxoj@protonmail.com
sox0j@protonmail.com
s0x0j@protonmail.com
50x0j@protonmail.com
...
```

[↑ Back to the start](#what-do-you-have)

## Other

- [Good random names generator](https://github.com/epidemics-scepticism/NickGenerator)

[↑ Back to the start](#what-do-you-have)
