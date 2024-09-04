# Helping FastAPI: How to contribute to docs translations

* [dev.to/ceb10n](https://dev.to/ceb10n/understanding-fastapi-how-fastapi-works-37od)
* [github.com/ceb10n](https://github.com/ceb10n/blog-posts/tree/master/helping-fastapi-how-to-contribute-to-docs-translations)

One of the great features of [FastAPI](https://fastapi.tiangolo.com/) is its great documentation 👏. But wouldn't it be better if more people around the world had access to this documentation? ❤️

Sometimes we take for granted that all people can read the docs we provide, but it's not necessarily how it works to people around the world.

## 📝 Why add translations?

📖 There is already a really good documentation written in english in FastAPI website, so why help translate to other languages?

Making a fast google search, you can see that only 17% of the 🌎 world population speaks english. You can check this wikipedia post: [List of countries by English-speaking population](https://en.wikipedia.org/wiki/List_of_countries_by_English-speaking_population) to check the percentage of english speakers in your country.

For example, I'm a brazilian living in Brazil. And here, only 5% of the population have some level of english. This represents a very small portion of the population that can follow documentation written in English.

And continuing with the portuguese, it's not only Portugal and Brazil that speaks this language. There's also Angola, Mozambique, Cape Verde and many other countries. You can see the [full list here](https://en.wikipedia.org/wiki/List_of_countries_and_territories_where_Portuguese_is_an_official_language).

Do you have any idea on how many people you can help when you translate the docs from your favorite programming language or framework? It's mindblowing thinking in the number of people who benefit from it.

Additionally, helping with translations is a very practical way to understand the way the project works, the workflow and approval that maintainers follow, etc.

## ✏️ How to create your first translation

[FastAPI docs](https://fastapi.tiangolo.com/) has a page dedicated on how to contribute to the project, including a section for the [documentations](https://fastapi.tiangolo.com/contributing/#docs) and [translations](https://fastapi.tiangolo.com/contributing/#translations).

So let's take a look step by step on how you can setup your local environment and start creating translations to a language other then english!

## 🍴 Forking FastAPI's repository

The first thing that you want to do is fork the FastAPI repo. [Github has a great doc](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo?platform=linux&tool=webui) explaining how you can fork a repository. But basically, what you'll need to do is browse the repo you want, in this case [FastAPI's repo](https://github.com/fastapi/fastapi) and click **Fork**.

![Fork FastAPI repository](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bi4ijicqhabiq7zx6gwt.jpg)

And that's it. Know you have your own copy of the repository. Now if you browse your own repositories, you'll see the forked repo right there:

![Forked repository](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/pheztl83unh62hwost7d.jpg)

## 🗂️ FastAPI's documentation structure

The FastAPI's documentation lives inside the folder docs, in the root of the repository. And all the source code from the docs lives inside docs_src folder.

![docs structure](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gjc8ubzxmrh7r46cst6x.png)

As you can see, inside docs folder, there are all the current languages that has translations. It uses the [ISO 693-1 two letter code](https://en.wikipedia.org/wiki/List_of_ISO_639_language_codes) for each language.

Each language folder will follow the same structure:

![Language translation structure](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vbfwr2mynbambri3oiit.png)

The `en` folder will have the complete documentation, but you'll notice that in other languages, despite having the same structure, not all files are present. And that's because not all files are translated to all languages (yet 😇).

💡 So now you know the first way to find what you can translate: There is a missing file in your language? That's the one you can start translating!

## ☹️ Missing languages

Not all languages has translations yet. For example, if you look for the 🇦🇲 Armenian code (hy) in the docs, you'll notice the it doesn't exist yet:

![Missing language](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/txr7msg4cdgsv53jfq7u.jpg)

In these cases, [FastAPI docs has a really good explanation](https://fastapi.tiangolo.com/contributing/#new-language) on how you can add translations to new languages.

As you can see from the docs, FastAPI has a neat script to initialize a new language translation:

```shell
python ./scripts/docs.py new-lang hy
```

Now that the script added the folder and files, you can follow the process of [adding translations to existing languages](https://fastapi.tiangolo.com/contributing/#existing-language).

## 🗺️ Translating existing languages

Now that you already forked the FastAPI's repository and learned how the add a missing language (if its your case), let's see the whole process of translating a file of the docs.

## 📋 Ways to verify what doc to translate

There are a few ways you can easily find what doc you can translate.

### 1️⃣ Navigating through the docs

A simple and easy way is: Read the docs in your desired language. Just go to the docs at [fastapi.tiangolo.com](https://fastapi.tiangolo.com/) and select the desired language:

![Desired language](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rzn4rt1d0xf5uz11wnz7.jpg)

Once you reach a doc that has no translations, you'll see a warning telling you that the doc has not been yet translated:

![doc not translated](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mdc77vpsjqxbphg429hk.jpg)

Now you can go to the source code, find the doc file and create a copy at your desired language folder. The folder structure of the docs are pretty simple to understand.

In the example above, the breadcrumbs say that we are at: FastAPI (The root) - Learn - Advanced. So we can infer that the document lives somewhere in docs/en/docs. Probably in some folder named learn or advanced.

If we go to the source code, will see that the folder advanced really exists, and the file `custom-response.md` also exists.

An easier way to find the file is get the last part of the url and find for it in your editor. For example, in VSCode you can use `ctrl + e` and enter that name:

![find the file in editor](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8bc8hxjfsm9poq0x256w.png)

### 2️⃣ Looking in source code

Another way you can find files that has no translations it to open the source code with your editor. Then you'll expand the desired language and the english language.

You will probably notice that the English folder has more files than the folder for your desired language. Now you can pick the missing file, create a copy of it at the language folder and translate it.

### 3️⃣ Using FastAPI Translations Management package

I created a 📦 package to help with FastAPI translations named [fastapi translations management](https://github.com/ceb10n/fastapi-translations-management).

It's basically a lib that will look at all doc files and check the last commit date. This will give you a list of files that has not been translated yet and the files that are outdated.

![fastapi translation management repo readme](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/v5xupb8qbhczaximxnno.jpg)

To use it, you can install it with [`pip`](https://pypi.org/project/fastapi-translations):

```shell
pip install -U fastapi-translations
```

And then use it at the root folder of your forked FastAPI repository:

```shell
fastapi_translations -l {two-letter-code-for-language}
```

This will give you a brief summary of missing translations and outdated translations:

![fastapi translation management translations summary](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qp4qyjv72laq97lkweko.jpg)

You can also generate a `csv` file with all files that are outdated and missing with `-c`:

```shell
fastapi_translations -l pt -c
```

## 📋 Things to know before start your first translation

### 💬 Discussions

If you want to translate to a language that already exists, probably it has a Topic created under Discussions. You can search your language in [github.com/fastapi/fastapi/discussions](https://github.com/fastapi/fastapi/discussions/categories/translations).

![Discussions for translations](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6x1teltu8xq2hnqay325.jpg)

At portuguese discussions, we have a pattern to always post the file we are translating, and after we finish, we edit it to add the PR link:

![Posts about translated docs](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/14puc1zhlu6vc7di4hak.jpg)

### 🔍 Reviews

All pull requests for translations must have at least two approvals from a person who speaks that language.

For example, if you create a translation for Japanese. Two people that speaks Japanese must review it before some mantainer approves the PR.

### 👨🏾‍⚖️ Another rules

There are some other rules that apply when we are translating some docs.

✅ Don't translate the code in `docs_src`;
✅ Only translate the markdown files;
✅ Inside code blocks, only translate the # comments;

You can check all the rules in FastAPI docs for [tips and guidelines for translations](https://fastapi.tiangolo.com/contributing/#translation-specific-tips-and-guidelines).

## ✨ Creating your first translation

Now that we know almost everything that is to be know about translating FastAPI docs, let's get started and translate a new doc.

### ⚙️ Update your FastAPI fork

Whenever you start a new translation, you need to update your forked repository to make sure everything is updated ✔️.

The easiest way to do this is to navigate to your repository at github and click in `sync fork -> update branch`.

![Update forked repository](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/q1bjlv044r263ql2q8jo.jpg)

Now you can update your local repository with all changes from the main repo.

![Pull master](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0toeiwnnqz8vv6teotxy.png)

### 🔎 Find the doc to translate

Now that our local repository is updated. Let's find some missing translation.

![Missing translation](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/03tiqj8ml2abvl8hqsw2.png)

We can see that under `docs/pt/docs/advanced`, the 📁 folder `security` is missing. So let's translate the `index.md` for the advanced security topic.

### 🙋🏿‍♀️ Notifying about translation in progress

Now that we picked a file to translate, let's tell everyone that in the 💬 Discussion for Portuguese translations that we are working on it:

![Notify the translation](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yhgpmc0azt0mmn6lnujr.jpg)

### 🛠️ Creating the translation

Not let's create a branch for the translation:

```shell
git checkout -b features/pt-advanced-security-index
```

Since we working on our local forked repository, we don't necessarily need to create a specific branch. But I think it's a good thing to do. And working this way, we can start another work while people are reviewing our PR.

Now we can create both the missing folder 📁, and the missing file 📃 under `docs/pt/docs/advanced`.

When I'm translating some file, I like to split the editor with the file that I'm working on, and the original file in english. But feel free to work the way is best for you.

![Translating the file](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/20dv61y6rx4w0t3g2f5w.png)

Now that we finished our work translating the file, we can commit it:

```shell
git add docs/pt/docs/advanced/security/index.md
git commit -m "Add translation to docs/pt/docs/advanced/security/index.md"
git push origin features/pt-advanced-security-index
```

### 👀 Previewing the translation

Now that we finished the translation, we can see how it will look like on the official docs.

You can type in your terminal 👨🏼‍💻 (remember to install all deps):

```shell
python scripts/docs.py live pt
```

And you'll be able to check the result:

![Preview of the translation](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kmls7ckqzmw8vsqir77t.jpg)

### 💻 Creating the Pull Request

Remember that we are working on our fork. Now that we commited to our repository, we need to send it to the [FastAPI repository](https://github.com/fastapi/fastapi). Luckily, this is very easy to do.

If you go to the [FastAPI repository](https://github.com/fastapi/fastapi), github will warn you that you pushed to your fork, and now you can create a PR to merge it:

![Github Warning to create a PR](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/svhshgf0wjw1vq3c2slp.jpg)

We can click on `compare & pull request` and create the PR following the pattern for the title:

> 🌐 Add Portuguese translation for `path/of/file.md`

![Creating the PR](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/7kp3f3g1iy9z1e56jzd6.jpg)

Now we can wait for all the checks to run (they must pass). And someone from the FastAPI team will add the necessary tags.

![Wait for the checks](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/na61vmavcmaprb9mugaz.jpg)

And of course, we need to update our post at the discussions to inform that we finished the translation:

![Update the discussion](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nooq4injkxm1jbga0hvd.jpg)

And after everything goes well, you'll get a message telling you that your PR was approved ✨:

![PR Approved](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/j4yid74j5gel0a6grogr.jpg)

### 💣 Dealing with problems

I didn't anticipate this when I started writing this article. A problem related with github actions and `upload-artifact` started happening and the checks from my PR failed 💥.

This was a really nice thing to happen to demonstrate how we can deal with situations that our PR has some problems.

![PR problem](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9rflzfllaff8gpgm516w.jpg)

When I saw the failing checks, I tried to see if it was related with my PR directly. I saw it was not related, and then I marked [Alejandra](https://github.com/alejsdev), who is a very helpful member of the [FastAPI team](https://fastapi.tiangolo.com/pt/fastapi-people/#team). [Sofie](https://github.com/svlandeg), who is also a member of the team mentioned the issue related with the problem right away.

![Issue related](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hdx8cmvrt28vfjd7rlp4.jpg)

As you can see, FastAPI has a really nice and helpful team and they always do their best to help you:

![FastAPI Team](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/epxxh97klsxbrmppmmnu.jpg)

So if you need some help, try to reach one the them. Just be polite and patiente that they will help you ❤️!

## 🎁 Benefits of translating docs

There are several benefits helping with the translations 🎉.

To me, the most important one is to help people who have difficulty reading documentation in english.

They can be 👩🏽‍🎓 students trying to learn a new language or framework, or even a 👩‍💻 professional who has not yet had the opportunity to learn english.

In addition to helping people, you can also 📖 learn new topics, find out about that detail that you used but didn't quite understand why, etc.

Also, helping translate docs requires you to review the original documentation. This may result in you finding improvements, topics that could be better explained, etc.

So, my advice is: do you have a language or framework that you really like and would like to start helping? Start with the documentation 📚!
