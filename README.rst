trustpaylib
===========

TrustPay payment solution integration helpers.


Install
-------

.. code:: bash

   $ pip install trustpaylib



Usage
-----


Create environment, payment request and generate signed link. 

.. code:: pycon

    >>> import trustpaylib
    >>> 
    >>> env = trustpaylib.build_environment(
    ...     aid="9876543210",
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


Or create initial data for form.
First merge payment request with environment variables, validate it and sign.
:func:`trustpaylib.TrustPay.finalize_request` will return prepared payment
request. And as form action use `trustpay_client.environment.api_url`.


.. code:: pycon

>>> # 
>>> pay_request = trustpay_client.finalize_request(pay_request)
 
>>> trustpay_client.initial_data(pay_request)
{'AID': '9876543210', 'REF': u'1234567890', 'AMT': u'123.45', 'SIG': 'DF174E635DABBFF7897A82822521DD739AE8CC2F83D65F6448DD2FF991481EA3', 'CUR': u'EUR'}

>>> trustpay_client.environment.api_url
'https://ib.trustpay.eu/mapi/paymentservice.aspx'
