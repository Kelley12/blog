---
title: "Financial Predictions"
layout: post
date: 2019-11- :: -0600
categories: posts
image: /assets/posts/.png
description: >
  
---

## Plaid setup guide

1. Set up your environment variables. [Plaid account keys](https://dashboard.plaid.com/account/keys)

    ```bash
    export PLAID_CLIENT_ID='************************'
    export PLAID_PUBLIC_KEY='******************************'
    export PLAID_SECRET='******************************'
    export PLAID_ENV='development' #or use ‘sandbox’ for the sample data
    ```

2. Clone the Plaid quickstart [repo](https://github.com/plaid/quickstart)

    ```bash
    git clone git@github.com:plaid/quickstart.git
    ```

3. Once the quickstart has been cloned, run:

    ```bash
    pip install -r requirements.txt
    ```

4. Edit server.py to include the environment variables you added in the first step

    ```python
    PLAID_CLIENT_ID = os.getenv('PLAID_CLIENT_ID')
    PLAID_SECRET = os.getenv('PLAID_SECRET')
    PLAID_PUBLIC_KEY = os.getenv('PLAID_PUBLIC_KEY')
    PLAID_ENV = os.getenv('PLAID_ENV')
    ```

5. Run server.py

    ```bash
    py server.py
    ```

6. Open up the following in your browser: [http://127.0.0.1:5000](http://127.0.0.1:5000)
7. Enter in your account credentials for your bank
8. Go back to the output from server.py, which should now have given you a token for the bank you just authorized
9. Set up the environment variable the same way you did in step 1 for the new bank token
export

    ```bash
    export CHASE_ACCESS_TOKEN='access-development-mysecrettoken'
    ```
