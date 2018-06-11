# Tutorial Setup
This tutorial assumes you have some basic knowledge of the concepts and usage of [`bagit`](https://tools.ietf.org/html/draft-kunze-bagit-16), [`bdbag`](https://github.com/fair-research/bdbag), and [`minid`](https://github.com/fair-research/minid). If not, click on the appropriate link(s) for additional information.

The following tutorial setup steps are common to all of the tutorials in this guide.
1. Ensure that you have the [prerequisites](#prerequisites) installed in your environment.
2. [Install](#installation) the software.
3. [Configure](#configuration) the software so that your own efforts are consistent with what is presented in the tutorials.

<a name="prerequisites"></a>
## Prerequisites
* Python 2.7 or higher
* The following Python modules: `setuptools`, `pip`, `wheel`. It is recommended to upgrade these modules even if they are already present. The following command can be used:
    ```sh
    pip install --upgrade pip setuptools wheel
    ```
* Optional: For uploading to AWS, you first need to install `awscli`, which you can install via `pip`.
Additionally, you need a valid AWS user account and need to know your `aws_access_key_id`
and `aws_secret_access_key_id`. Once you have these, run `aws configure` and enter the keys
at the prompts. Finally, create a bucket where you will upload your files.
You can do this via either `awscli` or the AWS Web Browser GUI.

Note: When using `pip install` commands, it may be necessary to use `sudo` on Unix and Mac
systems if you are modifying the system Python installation. Alternatively, you can pass the
`--user` flag to `pip install` in order to use the Python user install directory for your platform.

<a name="installation"></a>
## Installation
Use `pip` to install `bdbag` version `1.3.1` or greater, and `minid` version `1.2.4` or greater from PyPi:
```sh
pip install --upgrade 'bdbag>=1.3.1'
pip install --upgrade 'minid>=1.2.4'
```
Next, check that the correct versions were installed:
```sh
nih-commons:mdarcy[~] bdbag --version
1.3.1
nih-commons:mdarcy[~] minid --version
1.2.4
```
Finally, use `minid` to register yourself as a user with the `minid` server. Check your email for the server reponse which will contain a code that you will need to create and update `minid` identifiers.
```sh
nih-commons:mdarcy[~] minid --register_user --email mdarcy@not-my-real-email.org --name mdarcy --orcid 0000-0003-2280-917X
2018-06-06 23:14:06,959 - INFO - Registering new user "mdarcy" with email "mdarcy@not-my-real-email.org" and orcid "0000-0003-2280-917X"
```

<a name="configuration"></a>
## Configuration
Once you have `bdbag` and `minid` installed, there are a couple of recommended configuration
steps that you can perform that will save you from having to pass the same parameters repeatedly
when using the CLI or equivalent API functions.

First, modify the `~/.bdbag/bdbag.json` (or `%userprofile%\.bdbag\bdbag.json` on Windows)
configuration file and add key-value pairs such as `Source-Organization`, `Contact-Name`,
`Contact-Email`, `Contact-Orcid` to the `bag_metadata` element of the configuration.
Note that none of these elements are _required_, however if you want them to be automatically
populated in the `bag-info.txt` metadata of bags you create, it is a good idea to add them now.
```sh
nih-commons:mdarcy[~] cat ./.bdbag/bdbag.json
{
  "bag_config": {
    "bag_algorithms": [
      "md5"
    ],
    "bag_metadata": {
      "BagIt-Profile-Identifier": "https://raw.githubusercontent.com/fair-research/bdbag/master/profiles/bdbag-profile.json",
      "Contact-Name": "mdarcy",
      "Contact-Orcid": "0000-0003-2280-917X"
    },
    "bag_processes": 1
  },
  "identifier_resolvers": [
    "n2t.net",
    "identifiers.org"
  ]
}
```

Next, add your `minid` user information to the `~/.minid/minid-config.cfg` file:
```sh
nih-commons:mdarcy[~] cat .minid/minid-config.cfg
[general]
minid_server: http://minid.bd2k.org/minid
username: mdarcy
email: mdarcy@not-my-real-email.org
orcid: 0000-0003-2280-917X
code: e4506f64-9090-451f-ab4e-1192705dd89f
```

You're all set!

Ready to continue to Tutorial [#1](tutorial-1.md)?
