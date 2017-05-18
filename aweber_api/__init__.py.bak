from urlparse import parse_qs

from aweber_api.base import (
    ACCESS_TOKEN_URL,
    APIException,
    API_BASE,
    AUTHORIZE_URL,
    AWeberBase,
    REQUEST_TOKEN_URL,
)
from aweber_api.collection import AWeberCollection
from aweber_api.entry import AWeberEntry
from aweber_api.oauth import OAuthAdapter
from aweber_api.response import AWeberResponse


class AWeberAPI(AWeberBase):
    """Base class for connecting to the AWeberAPI.

    Created with a consumer key and secret, then used to either generate
    tokens for authorizing a user, or can be provided tokens and used to
    access that user's resources.

    """

    def __init__(self, consumer_key, consumer_secret):
        self.adapter = OAuthAdapter(consumer_key, consumer_secret, API_BASE)
        self.adapter.user = AWeberUser()

    @classmethod
    def parse_authorization_code(cls, authorization_code):
        """Exchange an authorization code for new api keys.

        Returns a tuple containing the new consumer key/secret and
        access token key/secret.

        """
        # parse and validate authorization code
        keys = cls._parse_and_validate_authorization_code(authorization_code)
        consumer_key = keys[0]
        consumer_secret = keys[1]

        # create an instance of AWeberAPI for getting the access token
        instance = cls._create_new_instance(keys)

        # exchange request token for an access token
        access_key, access_secret = instance.get_access_token()

        # return consumer key/secret and access token key/secret
        return consumer_key, consumer_secret, access_key, access_secret

    @classmethod
    def _parse_and_validate_authorization_code(cls, authorization_code):
        """parse and validate authorization code."""
        keys = authorization_code.split('|')
        if len(keys) < 5:
            raise APIException('Invalid Authorization Code')

        return keys

    @classmethod
    def _create_new_instance(cls, keys):
        """Create an instance of AWeberAPI for getting the access token."""
        instance = cls(keys[0], keys[1])
        instance.user.request_token = keys[2]
        instance.user.token_secret = keys[3]
        instance.user.verifier = keys[4]

        return instance

    @property
    def authorize_url(self):
        """Return the authorize url.

        Potentially containing the request token parameter.

        """
        if self.user.request_token:
            return "{0}?oauth_token={1}".format(
                AUTHORIZE_URL, self.user.request_token)

        return AUTHORIZE_URL

    def get_request_token(self, callback_url):
        """Get a new request token / token secret for the callback url.

        Returns request token / secret, and sets properties on the
        AWeberUser object (self.user).

        """
        data = {'oauth_callback': callback_url}
        response = self.adapter.request(
            'POST', REQUEST_TOKEN_URL, data)
        self.user.request_token, self.user.token_secret = (
            self._parse_token_response(response))

        return (self.user.request_token, self.user.token_secret)

    def get_access_token(self):
        """Exchange request tokens for Access tokens.

        Gets an access token for the combination of
        * request token
        * token secret
        * verifier
        in the AWeberUser object at self.user.

        Updates the user object and returns the tokens.

        """
        data = {'oauth_verifier': self.user.verifier}
        response = self.adapter.request(
            'POST', ACCESS_TOKEN_URL, data)
        self.user.access_token, self.user.token_secret = (
            self._parse_token_response(response))

        return (self.user.access_token, self.user.token_secret)

    def _parse_token_response(self, response):
        """Parses token response.

        Return the token key and the token secret

        """
        if not isinstance(response, str):
            raise TypeError('Expected response to be a string')

        data = parse_qs(response)

        if (data.get('oauth_token') is None) or (
                data.get('oauth_token_secret') is None):
            raise ValueError('OAuth parameters not returned')

        return (data['oauth_token'][0], data['oauth_token_secret'][0])

    def get_account(self, access_token=False, token_secret=False):
        """Returns the AWeberEntry object for the account.

        Specified by the access_token and token_secret currently
        in the self.user object.

        Optionally, access_token and token_secret can be provided to
        replace the properties in self.user.access_token and
        self.user.token_secret, respectively.

        """
        if access_token:
            self.user.access_token = access_token
        if token_secret:
            self.user.token_secret = token_secret

        url = '/accounts'
        response = self.adapter.request('GET', url)
        accounts = self._read_response(url, response)

        return accounts[0]


class AWeberUser(object):
    """Data storage object representing the user in the OAuth model.

    Has properties for request_token, token_secret, access_token, and
    verifier.

    """
    request_token = None
    token_secret = None
    access_token = None
    verifier = None

    def get_highest_priority_token(self):
        """Return either the access token or the request token."""
        return self.access_token or self.request_token
