# ansible-googleauth

![Build Status](http://bondi.local:3001/api/badges/Halfwalker/ansible-googleauth/status.svg)

This role is to install google authenticator and integrate it into ssh so that TOTP tokens may be required for ssh connections.

It will create a `~/.google_authenticator` if required, and will NOT alter or remove any existing version.

It will update `/etc/ssh/sshd_config.d` to ensure that a token is required for any ssh connection without an ssh key.  Connections _with_ an ssh key will not require a token, though this may be enabled so that tokens are *always* required.  Set the global **google_auth_force** variable to _true_ or an individual host entry (see below) to enable this.

## Configuration

To pre-populate the TOTP secret there are two locations to place the information.

* Place them into `defaults/main.yml` under the **google_auth_config** variable
* *Much* more preferably place them into an ansible-vault encrypted file under the **vault_google_auth_config** variable.  Typically this might be in `group_vars/all/vault`

The format is as follows
| Variable | Description |
| :--- | :--- |
| name: | The inventory_hostname for this block |
| force_auth: | force token for ALL ssh connections for this host |
| secret: | Standard `.google_authenticator` secret info

```yaml
# 1st line of secret can be 16 or 26 chars
vault_google_auth_config:
  - name: host1.example.com
    force_auth: false
    secret: |
      6DRWZ2AWOAFAQMSI
      "RATE_LIMIT 3 30
      " WINDOW_SIZE 3
      " DISALLOW_REUSE
      " TOTP_AUTH
      36011504
      52878834
      36710801
      23387673
      16670568
  - name: hosty.somewhere.com
    force_auth: false
    secret: |
      MVXECANUVTIQ2647HK3S35FM3A
      " RATE_LIMIT 3 30 1734051365
      " DISALLOW_REUSE 57801712
      " TOTP_AUTH
      17029728
      27355189
      27432004
      50794981
      18624382
```

NOTE: There must be at least 5x scratch codes for each secret

## Manually generating a google authenticator secret

For pre-populating the host secrets in the config, one can generate them via the command-line.  For an Ubuntu system, ensure the following packages are installed :

* libpam-google-authenticator
* python3-qrcode
* qrencode

Generate an authenticator secret, placed in `/tmp/google.txt`

```bash
google-authenticator --time-based --disallow-reuse --label=Test1 --qr-mode=UTF8 --rate-limit=3 --rate-time=30 --secret=/tmp/google.txt --window-size=3 --force
```

The contents of the resulting `/tmp/google.txt` may be placed directly into the `vault_google_auth_config` variable for a specific host.

## Methods of installation

There are several methods of creating the `~/.google_authenticator` file

* Existing `~/.google_authenticator`, no pre-config
    Any existing configuration will not be touched
* Existing `~/.google_authenticator`, pre-config in **vault_google_auth_config**
    Any existing configuration will not be touched.  You must manually remove any existing `~/.google_authenticator`
* No existing `~/.google_authenticator`, no pre-config
    In this case a new secret key and scratch codes will be created
* No existing`~/.google_authenticator`, pre-config in **vault_google_auth_config**
    If an entry in **vault_google_auth_config** exists it will be used, otherwise a new secret key will be created

