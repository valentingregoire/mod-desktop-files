# Modep/MOD on Raspberry PI 5

## Installation

### Modep

Initially, I wanted MOD Desktop to be running on my machine, but I did not get
it to build properly on/for the aarch64/arm64 architecture, even though there
are clear pointers in the source code that it should be supported. My lack of
knowledge might be the culprit here.

First, we need to install jack and get it working:

I ended up installing Modep via its `apt` repository
([sources](https://github.com/BlokasLabs/modep-debs)), which is the new way of
installing:

1. create keyring directory:

   ```bash
   sudo mkdir -p /etc/apt/keyrings
   ```

2. download the key:

   ```bash
   curl -fsSL https://blokas.io/apt/keys/blokas.gpg \
   | sudo gpg --dearmor -o /etc/apt/keyrings/blokas.gpg

   ```

3. add the `apt` repository:

   ```bash
   echo "deb [signed-by=/etc/apt/keyrings/blokas.gpg] http://blokas.io/apt/ buster main" \
   | sudo tee /etc/apt/sources.list.d/blokas.list

   ```

4. install Modep:

   ```bash
   sudo apt update
   sudo apt install modep
   ```

This will install Modep and all its dependencies.

### mod-ui

I couldn't get mod-ui running straight out of the box, so I had to improvise and
succeeded by building and running from the `modep-mod-ui` repo. This works from
the official `mod-ui` repo as well.

1. Clone the
   [git repo](https://github.com/BlokasLabs/mod-ui/tree/modep-1.13-ps):

   ```bash
   git clone git@github.com:BlokasLabs/mod-ui.git
   ```

2. Inside the repo, follow the
   [install section](git@github.com:BlokasLabs/mod-ui.git) of the readme, but
   with these remarks:
3. install Python 3.10 (with PyEnv):

   ```bash
   pyenv install 3.10
   ```

4. create a virtual environment based on Python 3.10:

   ```bash
   virtualenv modep-modui-venv
   source modep-modui-venv/bin/activate
   ```

5. install the dependencies, but with version `3.4` for . Therefore, adapt the
   file `requirements.txt` and put this change the version of to 3.4:

   ```bash
   ==3.4
   ```

   Now install: `bash   pip install -Ur requirements.txt   `

   Since we are using Python 3.10, we need to hack a little because is not
   supported anymore in Python 3.10. Instead, we need to use the `abc` version
   instead:

   ```bash

   ```

6. create a file called `.env` where we will store some environmental variables
   that need to be set in order to let mopdep-mod-ui interact with
   modep-mod-host:

7. run the server:

   ```bash
   source modep-modui-venv/bin/activate
   source .venv
   python ./server.py
   ```
