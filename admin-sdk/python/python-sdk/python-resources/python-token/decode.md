# decode

Decodes a [DID Token](../../../../../decentralized-id.md) from a Base64 string into a tuple of its individual components: `proof` and `claim`. This method allows you decode the DID Token and inspect the token. You can apply your own rules and validations on top of the current [Token.validate](validate.md) method. 

```text
Token.decode(did_token)
```

### Arguments:

* `did_token` \(str\): A [DID Token](../../../../../decentralized-id.md) generated by a Magic user on the client-side.

### Raises:

* `DIDTokenError` if the given DID Token is malformed.

### Returns:

* `proof` \(str\): A digital signature that proves the validity of the given `claim`
* `claim` \(dict\): Unsigned data the user asserts. This should equal the `proof` after Elliptic Curve recovery. See [Decentralized ID Token Specification](../../../../../decentralized-id.md#decentralized-id-token-specification) for fields inside the `claim`.

### Examples:

```python
from magic_admin import Magic
# A util provided by `magic_admin` to parse the auth header value.
from magic_admin.utils.http import parse_authorization_header_value
from magic_admin.error import DIDTokenError


# An exmaple of user info view.
@user.route('/v1/user/info', method=['GET'])
def user_info(self):
    """An exmaple of user info view that uses DID Token to ensure
    authenticity of a request before returning user info.
    """
    # Parse the `Authorization` header value for did_token.
    did_token = parse_authorization_header_value(
        requests.headers.get('Authorization'),
    )
    if did_token is None:
        raise BadRequest(
            'Authorization header is missing or header value is invalid',
        )
    
    magic = Magic(api_secret_key='<YOUR_API_SECRET_KEY>')

    # Validate and decode the token.
    try:
        magic.Token.validate(did_token)
        proof, claim = magic.Token.decode(did_token)
    except DIDTokenError as e:
        raise BadRequest('DID Token is malformed: {}'.format(e))
    
    # You can perform your own validations on the claim fields.
    if claim['iss'] != 'troll_goat' and claim['aud'] != 'app':
        raise BadRequest()
    
    # Use your application logic to load the user info.
    user_info = logic.User.load_by(issuer=issuer)
    
    return HttpResponse(user_info)
```

{% hint style="warning" %}
It is important to always validate the DID Token before using.
{% endhint %}

