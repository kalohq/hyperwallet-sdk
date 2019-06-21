This is a fork of https://pypi.python.org/pypi/hyperwallet-sdk/0.2.0 which was
the only source code available at the time.

Hyperwallet has since published on github
(https://github.com/hyperwallet/python-sdk) so we should clean up our fork and
make a pull request against the official repo.

---

# Basics

## Setting up

Development & Testing is done within a virtualenv

```
virtualenv .venv
source .venv/bin/activate
```

## Running tests

```
python setup.py test
```

## Building

```
python setup.py build
```

## Installing

```
python setup.py install
```

## Usage

<details>
<summary>Expand ...</summary>

### Create `hw.py`

```
import json
from datetime import datetime, timedelta
import hyperwallet

# UAT
HYPERWALLET_USERNAME = 'XXX'
HYPERWALLET_PASSWORD = 'XXX'
HYPERWALLET_SERVER = 'https://hyperwallet-uat.preproduction.kalohq-infra.net'

api = hyperwallet.Api(
    HYPERWALLET_USERNAME,
    HYPERWALLET_PASSWORD,
    HYPERWALLET_SERVER,
)

def find_user(email):
    params = {'email': email}
    response = api.listUsers(params)
    print(json.dumps(response['data'], indent=2))


def list_all_users(count=100):
    params = {'limit': str(count)}
    response = api.listUsers(params)
    print(json.dumps(response['data'], indent=2))


def list_user_bank_accounts(user_token):
    bank_res = api.listBankAccounts(user_token)
    for bank in bank_res['data']:
        print(json.dumps(bank, indent=2))


def fetch_bank_account(user_token, bank_token):
    response = api.retrieveBankAccount(user_token, bank_token)
    print(json.dumps(response, indent=2))


def deactivate_bank_account(user_token, bank_token):
    response = api.createBankAccountStatusTransition(user_token, bank_token, {
        'transition': 'DE_ACTIVATED',
        'notes': 'Deleting bank account upon user\'s request'
    })
    print(json.dumps(response, indent=2))


def list_payments():
    # fetch all payments, iterating through API pagination
    limit = 100
    offset = 0
    count = None
    cutoff_date = (datetime.utcnow() - timedelta(days=30)).isoformat()
    while offset < count or count is None:
        response = api.listPayments(params={
            'limit': str(limit), 'offset': str(offset),
            'createdAfter': cutoff_date,
            'sortBy': '-createdOn',
        })
        offset = offset + len(response['data'])
        count = response['count']
        for payment in response['data']:
            print('{} - {} - {}'.format(
                p['createdOn'], p['clientPaymentId'], p['status']
            ))


list_all_users()

```

### Run the code

```
pip install https://github.com/kalohq/hyperwallet-sdk/archive/v0.3.6.zip
export HYPERWALLET_DEBUG=1  # log all API requests/responses to stdout
python hw.py
```
</details>

