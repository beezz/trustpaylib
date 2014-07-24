trustpaylib
===========

.. image:: https://travis-ci.org/beezz/trustpaylib.svg?branch=master
    :target: https://travis-ci.org/beezz/trustpaylib

.. image:: https://coveralls.io/repos/beezz/trustpaylib/badge.png
   :target: https://coveralls.io/r/beezz/trustpaylib 


TrustPay payment solution integration helpers.


Install
-------

.. code:: bash

   $ pip install trustpaylib



Usage
-----


Link approach
.............

Create environment, payment request and generate signed link. 

.. code:: pycon

    >>> import trustpaylib
    >>> 
    >>> env = trustpaylib.build_environment(
    ...     aid="1234567890",
    ...     secret_key="abcd1234",
    ...     # api_url=trustpaylib.TRUSTCARD_API_URL,
    ... )
    >>> pay_request = trustpaylib.build_pay_request(
    ...     AMT="123.45",
    ...     CUR="EUR",
    ...     REF="1234567890",
    ... )
    >>> trustpay_client = trustpaylib.TrustPay(env)
    >>> trustpay_client.build_link(pay_request)
    'https://ib.trustpay.eu/mapi/paymentservice.aspx?AID=9876543210&REF=1234567890&AMT=123.45&SIG=DF174E635DABBFF7897A82822521DD739AE8CC2F83D65F6448DD2FF991481EA3&CUR=EUR'



Form approach
.............

First merge payment request with environment variables, validate it and sign.
`trustpaylib.TrustPay.finalize_request` returns prepared payment
request. As form action use `trustpay_client.environment.api_url`.


.. code:: pycon

    >>> pay_request = trustpay_client.finalize_request(pay_request)
    >>> trustpay_client.initial_data(pay_request)
    {'AID': '9876543210', 'REF': u'1234567890', 'AMT': u'123.45', 'SIG': 'DF174E635DABBFF7897A82822521DD739AE8CC2F83D65F6448DD2FF991481EA3', 'CUR': u'EUR'}
    >>> trustpay_client.environment.api_url
    'https://ib.trustpay.eu/mapi/paymentservice.aspx'


Redirects handling
..................

There's not much you can (or should) do with redirect.

.. note::

     Official TrustPay documentation: DO NOT PERFORM ANY ACTION ON THIS REDIRECT.
     Data is not signed and therefore cannot beconsidered as a verified payment result, such
     as the signed results sent to Notification URL or NotificationEmail.


But at least you can check the result code.


.. code:: pycon

    >>> # data received as get params to redirect
    ... redirect_data = {"REF": "1234567890", "RES": 3, "PID": "1212321"}
    >>> redirect = trustpaylib.build_redirect(**redirect_data)
    >>> 
    >>> trustpay_client.get_result_desc_from_redirect(redirect).short
    'Authorized'
    >>> trustpay_client.get_result_desc_from_redirect(redirect).long
    'Payment was successfully authorized. Another notification (with result code 0 - success) will be sent when TrustPay receives and processes payment from 3rd party.'



Notifications handling
......................


For received notification, first check signature.

.. code:: pycon

    >>> notification_data = {
    ...     "AID": "1234567890",
    ...     "TYP": "CRDT",
    ...     "AMT": "123.45",
    ...     "CUR": "EUR",
    ...     "REF": "9876543210",
    ...     "RES": "0",
    ...     "TID": "11111",
    ...     "OID": "1122334455",
    ...     "TSS": "Y",
    ...     "SIG": (
    ...         "97C92D7A0C0AD99CE5DE55C3597D5ADA"
    ...         "0D423991E2D01938BC0F684244814A37"
    ...     ),
    ... }
    >>> notification = trustpaylib.build_notification(**notification_data)
    >>> assert trustpay_client.check_notification_signature(notification)


Then check result code.


.. code:: pycon

    >>> trustpay_client.get_result_desc_from_notification(notification).short
    'Success'
    >>> trustpay_client.get_result_desc_from_notification(notification).long
    'Payment was successfully processed.'
