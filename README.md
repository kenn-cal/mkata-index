<h1 align="center">github-hosted-pypi</h1>

<p align="center">
By Nicolas Remond, adapted for M'kata, a project by Kenn Gerard Cal
</p>

---

<p align="center">
  <a href="#description">Description</a> •
  <a href="#features">Features</a> •
  <a href="#get-started">Get Started</a> •
  <a href="#modify-indexed-packages">Modify indexed packages</a> •
  <a href="#faq">FAQ</a> •
  <a href="#a-word-about-supply-chain-attacks">A word about supply chain attacks</a> •
  <a href="#contribute">Contribute</a> •
  <a href="#references">References</a>
</p>

---

## Features

* **😺 GitHub-hosted**
* **🚀 Template ready to deploy**
* **🔆 Easy to use** through GitHub Actions
* **🚨 Secure**: Warns you if your package is vulnerable to supply chain attacks

## Description

This repository is a GitHub page used as a PyPi index, conform to [PEP503](https://www.python.org/dev/peps/pep-0503/).

You can use it to group all your packages in one place, and access it easily through `pip`, almost like any other
package publicly available.

---

_While the PyPi index is public, private packages indexed here are kept private: you will need GitHub authentication to
be able to retrieve it._

## Features

Visit [the index](https://kenn-cal.github.io/mkata-index/) and try to install packages indexed there!

---

Install the core package `mkata`:
```console
pip install mkata-core --extra-index-url https://kenn-cal.github.io/mkata-index/
```

It will also install the dependencies automagically.

You can also install a specific version:

```console
pip install mkata-core==0.0.1 --extra-index-url https://kenn-cal.github.io/mkata-index/
```

All packages are private; only those with rights will be able to access them.

## Get started

* Use this template and create your own repository:

<p align="center">
    <a href="https://github.com/astariul/github-hosted-pypi/generate">
        <img
            src="https://img.shields.io/badge/%20-Use%20this%20template-green?style=for-the-badge&color=347d39"
            alt="Use template" />
    </a>
</p>

* Go to `Settings` of your repository, and enable GitHub Page
* Customize `index.html` and `pkg_template.html` to your liking
* You're ready to go! Visit `<user>.github.io/<repo_name>` to see your PyPi index

## Modify indexed packages

Now that your PyPi index is set up, you can register / update / delete packages indexed.  
_GitHub actions are set up to do it automatically for you._

You just have to:
* Go to the `Actions` tab of your repository
* Click the right workflow (`register` / `update` / `delete`) and trigger it manually
* Fill the form and start the workflow
* Wait a bit
* Check the new PR opened (ensure the code added correspond to what you want)
* Merge the PR

## FAQ

#### Q. Is it secure?

As you may know, `pip` can install GitHub-hosted package if given in the form `pip install git+<repo_link>`. This PyPi
index is just an index of links to other GitHub repository.

GitHub Pages are public, so this PyPi index is public. But it just contains links to other GitHub repositories, no code
is hosted on this PyPi index.

If the repository hosting code is private, you will need to authenticate with GitHub to be able to clone it, effectively
making it private.

---

If you wonder more specifically about supply chain attacks, check [the section about it]
(#a-word-about-supply-chain-attacks).

#### Q. What happens behind the scenes?

When running `pip install <package_name> --extra-index-url https://astariul.github.io/github-hosted-pypi/`, the
following happen:

1. `pip` will look at `https://pypi.org/`, the default, public index, trying to find a package with the specified name.
2. If it can't find, it will look at `https://astariul.github.io/github-hosted-pypi/`.
3. If the package is found there, the link of the package is returned to `pip` (`git+<repo_link>@<tag>`).
4. From this link, `pip` understands it's a GitHub repository and will clone the repository (at the specific tag) locally.
5. From the cloned repository, `pip` install the package.
6. `pip` install any missing dependency with the same steps.

_Authentication happens at step 4, when cloning the repository._

#### Q. What are the best practices for using this PyPi index?

The single best practice is using GitHub releases. This allows your package to have a version referred by a specific
tag. To do this:

* Push your code into a repository.
* Create a new GitHub release. Ensure you follow [semantic versioning](https://semver.org/). It will create a tag.
* Ensure you can install the package with `pip install git+<repo_link>@<tag>`
* When putting the package on this index, put the full link (`git+<repo_link>@<tag>`).

#### Q. What if the name of my package is already taken by a package in the public index?

You can just specify a different name for your indexed package. Just give it a different name in the form when
registering it.

For example if you have a private package named `tensorflow`, when you register it in this index, you can name it
`my_cool_tensorflow`, so there is no name-collision with the public package `tensorflow`.  

Then you can install it with:
`pip install my_cool_tensorflow --extra-index-url https://astariul.github.io/github-hosted-pypi/`.

Then from `python`, you can just do:
```python
import tensorflow
```

---

**But be careful!** While it's possible to handle it like this, it's always better to have a unique name for your
package, to avoid confusion but also for [security](#a-word-about-supply-chain-attacks)!

#### Q. How to download private package from a Docker?

Building a Docker image is not interactive, so there is no prompt to type username and password.  
Instead, you should put your GitHub credentials in a `.netrc` file, so `pip` can authenticate when cloning from GitHub.  
To do this securely on Docker, you should use Docker secrets. Here is a quick tutorial on how to do:

**Step 1**: Save your credentials in a secret file. Follow this example:

```
machine github.com
	login <gh_user>
	password <gh_pass>
```

⚠️ _Syntax is important: ensure you're using **tabulation**, and the line endings are **`\n`**.  
So careful if you're using an IDE that replace tabs by spaces or if you're on Windows (where line endings are `\r\n`)!_

Let's name this file `gh_auth.txt`.

**Step 2**: Create your Docker file. In the docker file you should mount the secret file in `.netrc`, and run the
command where you need authentication. For example:

```dockerfile
# syntax=docker/dockerfile:experimental
FROM python:3

RUN --mount=type=secret,id=gh_auth,dst=/root/.netrc pip install <package_name> --extra-index-url https://astariul.github.io/github-hosted-pypi/
```

**Step 3**: Build your Docker image, specifying the location of the secret created in step 1:

`sudo DOCKER_BUILDKIT=1 docker build --secret id=gh_auth,src=./gh_auth.txt .`

---

**_If you have any questions or ideas to improve this FAQ, please open a PR / blank issue!_**

## A word about supply chain attacks

As you saw earlier, this github-hosted PyPi index rely on the `pip` feature `--extra-index-url`. Because of how this
feature works, it is vulnerable to supply chain attacks.

For example, let's say you have a package named `fbi_package` version `2.8.3` hosted on your private PyPi index.

An attacker could create a malicious package with the same name and a higher version (for example `99.0.0`).  
When you run `pip install fbi_package --extra-index-url my_pypi_index.com`, under the hood `pip` will download the
latest version of the package, which is the malicious package!

---

While this repository makes it very convenient to have your own PyPi index, be aware that the page is public, therefore
anyone can see which package name you're using and create a malicious package with this same name.

That's why we included automated checks into this private PyPi index. Whenever you access the page of your package, PyPi
API is called, and if a package with the same name and a higher version is found, the `install` command is replaced with
a warning.

You can see a demo of such warning at [https://astariul.github.io/github-hosted-pypi/transformers/](https://astariul.github.io/github-hosted-pypi/transformers/).

If you see this warning, don't install the package! Instead, change the name of your package or upgrade the version
above its public counterpart.

Be careful out there!

## Contribute

Issues and PRs are welcome!

If you come across anything weird / that can be improved, please get in touch!

## References

**This is greatly inspired from [this repository](https://github.com/ceddlyburge/python-package-server).**  
It's just a glorified version, with cleaner pages and GitHub actions for easily adding, updating and removing packages
from your index.

Also check the [blogpost](https://www.freecodecamp.org/news/how-to-use-github-as-a-pypi-server-1c3b0d07db2/) of the original author!

---

_Icon used in the page was made by [Freepik](https://www.flaticon.com/authors/freepik) from [Flaticon](https://www.flaticon.com/)_
