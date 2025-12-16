# Modep/MOD on Raspberry PI 5 & HifiBerry Studio DAC/ADC XLR

## General info

- [HifiBerry Studio DAC/ADC XLR info](https://www.hifiberry.com/shop/boards/hifiberry-studio-dacadc-xlr/)
- [HifiBerry Studio DAC/ADC XLR datasheet](https://www.hifiberry.com/docs/data-sheets/datasheet-studio-dac-adc/)

## Installation

### Modep

Initially, I wanted MOD Desktop to be running on my machine, but I did not get
it to build properly on/for the aarch64/arm64 architecture, even though there
are clear pointers in the source code that it should be supported. My lack of
knowledge might be the culprit here.

First, we need to install jack and get it working.

### Jack

It's best if Jack is run via `jack_control`, because then we have control over
it with `qjackctl`, which is very useful. This is the command I use that works:

```bash
/usr/bin/jack_control start
```

I had to set all sorts of parameters to make sure it works. This is the
equivalent command that works for jack. Try to replicate all the parameters via
`jack_control`:

```bash
/usr/bin/jackd -t 2000 -R -d alsa -dhw:sndrpihifiberry,0 -r 48000 -p 128 -n 2 -X seq -s -S -i 2 -o 2
```

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

5. install the dependencies, but with version `3.4` for `pyserial`. Therefore,
   adapt the file `requirements.txt` and put this change the version of to
   `3.4`:

   ```bash
   pyserial==3.4
   ```

   Now install:

   ```bash
   pip install -Ur requirements.txt
   ```

   Since we are using Python 3.10, we need to hack a little because
   `collections.MutableMapping` is not supported anymore in Python 3.10.
   Instead, we need to use the `abc` version instead:

   ```bash
    sed -i -e 's/collections.MutableMapping/collections.abc.MutableMapping/' \               git:main*
      modep-modui-env/lib/python3.10/site-packages/tornado/httputil.py

   ```

6. create a file called `.env` where we will store some environmental variables
   that need to be set in order to let modep-mod-ui interact with
   modep-mod-host:

   ```bash
   export MOD_DEV_HOST=0
    export MOD_DEV_ENVIRONMENT=0
    export LV2_PATH=/var/modep/lv2:/var/modep/lv2-presets
    export LV2_PLUGIN_DIR=/var/modep/lv2
    export MOD_USER_PEDALBOARDS_DIR=/var/modep/pedalboards
    export MOD_DEV_ENVIRONMENT=0
    export MOD_DEVICE_WEBSERVER_PORT=8888
    export MOD_LOG=1
    export MOD_APP=0
    export MOD_LIVE_ISO=0
    export MOD_SYSTEM_OUTPUT=1
    export MOD_DATA_DIR=/var/modep
    export MOD_PRESETS_DIR=/var/modep/lv2-presets
    export MOD_USER_FILES_DIR=/var/modep/user-files
    #export JACK_PROMISCUOUS_SERVER=jack
    export PATCHSTORAGE_API_URL=https://patchstorage.com/api/beta/patches
    export PATCHSTORAGE_PLATFORM_ID=8046
    export PATCHSTORAGE_AARCH64_TARGET_ID=8280
    export PATCHSTORAGE_ARMHF_TARGET_ID=8278
    #export MOD_KEY_PATH=/var/modep/keys
    #export MOD_API_KEY=/var/modep/mod_api_key.pub
    export MOD_HTML_DIR=/usr/share/mod/html
    export MOD_DEFAULT_PEDALBOARD=/usr/share/mod/default.pedalboard
   ```

7. run the server:

   ```bash
   source modep-modui-venv/bin/activate
   source .env
   python ./server.py
   ```
