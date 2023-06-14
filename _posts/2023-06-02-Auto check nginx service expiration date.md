---
title: Auto check nginx service expiration date
date: 2023-06-02 11:11:00 -0700
categories: [Nginx]
tags: 
---

Sometimes, I find myself needing to manage my nginx service, especially when it's close to the service expiration, and we need to connect with customers. However, we tend to forget about it, so I made a decision to create a script that runs on the server, automating the management process.

Here's the script I've developed:

1. #### Nginx_expiration.sh

we can use this scipt check expiration and sen SMS to customers.

```shell
#!/bin/bash

# * Vhost config file path
vhost_config_path="/etc/nginx/conf.d"

# * Aliyun SMS script path
sms_script_path="/home/workspace/aliyun_sms/send_sms.py"

# * Python virtualenv path(optional)
python_virtualenv_path="/home/workspace/aliyun_sms/.aliyun_sms/bin/activate"

# * Log file path
log_file_path="/home/workspace/aliyun_sms/nginx_expiration_check.log"

# * Get current date in YYYY-MM-DD format
current_date=$(date +%Y-%m-%d)

# * Source Python virtualenv
source "$python_virtualenv_path"

# * For loop to iterate through all vhost config files in the vhost config path
for vhost_config_file in $(ls "$vhost_config_path"/*.conf); do
    # * Pick out the vhost name from the vhost config file name
    vhost_name=$(basename "$vhost_config_file" .conf)

    # * GET phone_number list
    phone_numbers=()
    # while IFS= read -r vhost_config_file; do
        while IFS= read -r line; do
            if [[ $line == *"# phone_number="* ]]; then
                phone_number=$(echo "$line" | sed -n 's/.*phone_number=\(.*\)/\1/p')
                phone_numbers+=("$phone_number")
                echo "Found phone number: $phone_number"
            fi
        done < "$vhost_config_file"
    done < <(find "$vhost_config_path" -name "*.conf")

    # * Get phone number from vhost config file(only one phone number)
    # phone_number=$(grep "phone_number=" "$vhost_config_file" | cut -d'=' -f2)
    # echo "Found phone number: $phone_number"

    # * Get expiration date from vhost config file
    expiration_date=$(grep "expiration_date=" "$vhost_config_file" | cut -d'=' -f2)
    days_remaining=$(( ($(date -d "$expiration_date" +%s) - $(date -d "$current_date" +%s)) / 86400 ))

    # * Send SMS if expiration date is 30, 15 or 7 days away
    if [[ $days_remaining -eq 30 || $days_remaining -eq 15 || $days_remaining -eq 7 ]]; then
        message="Your virtual host $vhost_name will expire in $days_remaining days."
        for phone_number in "${phone_numbers[@]}"; do
            python "$sms_script_path" "$phone_number" "$vhost_name" "$days_remaining"
        done
        # python "$sms_script_path" "$phone_number" "$vhost_name" “$days_remaining”
        echo "$(date +%Y-%m-%d) - Vhost: $vhost_name, Days Remaining: $days_remaining" >> "$log_file_path"
    fi

    # * Disable vhost if expiration date is 0 days away, just comment out the listen directives
    if [[ $current_date > $expiration_date ]]; then
        sed -i 's/listen 80;/#listen 80;/g' "$vhost_config_file"
        sed -i 's/listen 443 ssl;/#listen 443 ssl;/g' "$vhost_config_file"
        nginx -s reload
    fi
```

2. #### Send_sms.py

We use Aliyun Python SMS SDK send message.

First, you should install the SDK

`pip install aliyun-python-sdk-core`

```python
import sys
import configparser
from aliyunsdkcore.client import AcsClient
from aliyunsdkcore.request import CommonRequest
from encrypt import decrypt_config, load_key
import datetime

# * Read config.ini
# config = configparser.ConfigParser()
# config.read('config.ini')

# * get aliyun sms config from config.ini file, this is not secure
# access_key = config.get('aliyun', 'access_key')
# secret_key = config.get('aliyun', 'secret_key')
# sign_name = config.get('aliyun', 'sign_name')
# template_code = config.get('aliyun', 'template_code')

# * Load key
key = load_key()

# * If use docker ENV. Retrieve the key from the environment variable $MY_KEY
# * Set up the key use 'docker run --env MY_KEY=$MY_KEY my-image'
# key = os.getenv('MY_KEY')

# * Get aliyun sms config from encrypted config file, this is secure

decrypted_config = decrypt_config('encrypted_config.ini', key)
access_key = decrypted_config.get('aliyun', 'access_key')
secret_key = decrypted_config.get('aliyun', 'secret_key')
sign_name = decrypted_config.get('aliyun', 'sign_name')
template_code = decrypted_config.get('aliyun', 'template_code')

phone_number = sys.argv[1]
vhost_name = sys.argv[2]
days_remaining = sys.argv[3]

client = AcsClient(access_key, secret_key, 'default')
request = CommonRequest()
request.set_domain('dysmsapi.aliyuncs.com')
request.set_version('2017-05-25')
request.set_action_name('SendSms')
request.set_method('POST')
request.add_query_param('SignName', sign_name)
request.add_query_param('TemplateCode', template_code)
request.add_query_param('PhoneNumbers', phone_number)
request.add_query_param('TemplateParam', '{"vhost_name":"' + vhost_name + '","days_remaining":"' + days_remaining + '"}')

response = client.do_action_with_exception(request)

# * Record log to sms_log.log with current time Phone number and response
current_time = datetime.datetime.now().strftime("%Y-%m-%d %H:%M")
log_message = f"{current_time}: Phone number: {phone_number}, Response: {response.decode()}"
with open('sms_log.log', 'a') as log_file:
    log_file.write(log_message + '\n')

sys.stdout.write(response.decode())

```

We use the Python cryptography encrypted the information with access key, secret key, etc.

Below is the encrypt and decrypt code, please don't forget `pip install cryptography` **this encrypt is optional.**

```python
import configparser
from cryptography.fernet import Fernet

# * Generate_key
def generate_key():
    key = Fernet.generate_key()
    with open('key.key', 'wb') as key_file:
        key_file.write(key)

# * Load_key
def load_key():
    with open('/home/workspace/key/key.key', 'rb') as key_file:
        key = key_file.read()
    return key

# * Encrypt_config to encrypted_config.ini
def encrypt_config(config_file, key):
    cipher_suite = Fernet(key)

    config = configparser.ConfigParser()
    config.read(config_file)

    encrypted_config = configparser.ConfigParser()
    for section in config.sections():
        encrypted_config.add_section(section)
        for option in config.options(section):
            value = config.get(section, option)
            encrypted_value = cipher_suite.encrypt(value.encode())
            encrypted_config.set(section, option, encrypted_value.decode())

    with open('encrypted_config.ini', 'w') as encrypted_file:
        encrypted_config.write(encrypted_file)

# * Decrypt_config
def decrypt_config(encrypted_file, key):
    cipher_suite = Fernet(key)

    encrypted_config = configparser.ConfigParser()
    encrypted_config.read(encrypted_file)

    config = configparser.ConfigParser()
    for section in encrypted_config.sections():
        config.add_section(section)
        for option in encrypted_config.options(section):
            encrypted_value = encrypted_config.get(section, option)
            decrypted_value = cipher_suite.decrypt(encrypted_value.encode())
            config.set(section, option, decrypted_value.decode())

    return config

# * Create key
# generate_key()

# * Load key
# key = load_key()

# * Encrypt config.ini
# encrypt_config('config.ini', key)

# * Decrypt config.ini
# decrypted_config = decrypt_config('encrypted_config.ini', key)
# print(decrypted_config.get('aliyun', 'access_key'))
# print(decrypted_config.get('aliyun', 'secret_key'))
# print(decrypted_config.get('aliyun', 'sign_name'))
# print(decrypted_config.get('aliyun', 'template_code'))
```

