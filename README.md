# yubikey-otp-sh
Method to validate Yubikey OTP codes via api.yubikey.com in bash

Modern Yubikeys can generate OTP codes.  They can be validated online
using the yubico APIs.  The protocol is described at 

  https://developers.yubico.com/OTP/Specifications/OTP_validation_protocol.html

However I sometimes find code easier to follow than verbose descriptions,
so I put together this proof of concept, in bash, to make it easy to
understand what is going on.

You need an ID and an API key for this to work (so you can verify the
HMAC of the results to ensure no tampering).  These can be got got free
from https://developers.yubico.com/OTP/

Basically the code does:

* Request the OTP from the user
* Generate a nonce (one time code; this helps prevent replays)
* Send the request to YubiCloud
* Parse the response, check the HMAC, verify the status

The response looks something like

    h=bMZpcTlVe7RPCsGmbSiM40iLfj4=
    t=2022-03-22T15:05:26Z0823
    otp=abcdefghijklhuhuitldienftdkdhhbntncldljtdlnj
    nonce=askjdnkajsndjkasndkjsnad
    sl=100
    status=OK

To generate the string to calculate the HMAC on we need to remove the h=
line, put the results in order and then put them onto one line seperated
by &

    nonce=...&otp=...&sl=...&status=...&t=...

That's what this line does (there may be better ways, but this works!)

    grep -v '^h=' | sort | tr '\012' '&' | sed "s/&$//"

If the HMAC doesn't match then we have a bad response from the server.

If the HMAC does match then we can check the status= value; anything except OK
is an error.

If all is good then the first 12 characters of the OTP string; these are
unique to each key and will always be there in every key press.

Example of a good request:

    % ./otp
    Press the button on your key: (+)
    Good code. Your ID is abcdefghijkl

Example of a replay (the same OTP code presented twice)

    % ./otp
    Press the button on your key: (+)
    Bad status code: REPLAYED_OTP
    h=aJvUXOsedgQcavf2HKaKgdUF5Ro=
    t=2022-03-23T13:22:43Z0131
    otp=abcdefghijklktbtbguhbffbcifgblregbkvhrugenlv
    nonce=jndlVdKyC85wfL7EWQEavySs
    status=REPLAYED_OTP

