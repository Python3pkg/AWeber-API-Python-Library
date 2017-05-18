from urllib import urlencode
import json
import os

import oauth2 as oauth

from aweber_api.base import APIException


class OAuthAdapter(object):

    def __init__(self, key, secret, base):
        self.key = key
        self.secret = secret
        self.consumer = oauth.Consumer(key=self.key, secret=self.secret)
        self.api_base = base

    def _parse(self, response):
        try:
            data = json.loads(response)
            if not data or data == '':
                return response
            return data
        except:
            pass
        return response

    def request(self, method, url, data={}, response='body'):
        client = self._get_client()
        url = self._expand_url(url)
        body = self._prepare_request_body(method, url, data)

        content_type = 'application/json'
        if method == 'GET' and body is not None and body is not '':
            if '?' in url:
                url = '{0}&{1}'.format(url, body)
            else:
                url = '{0}?{1}'.format(url, body)

        if method == 'POST':
            content_type = 'application/x-www-form-urlencoded'
        headers = {'Content-Type': content_type}

        resp, content = client.request(
            url, method, body=body, headers=headers)

        if int(resp['status']) >= 400:
            """
            API Service Errors:

            Please review the Exception that is raised it should indicate
            what the error is.

            refer to https://labs.aweber.com/docs/troubleshooting for more
            details.
            """
            content = json.loads(content)
            error = content.get('error', {})
            error_type = error.get('type')
            error_msg = error.get('message')
            raise APIException(
                '{0}: {1}'.format(error_type, error_msg))

        if response == 'body' and isinstance(content, str):
            return self._parse(content)
        if response == 'status':
            return resp['status']
        if response == 'headers':
            return resp
        return None

    def _expand_url(self, url):
        if not url[:4] == 'http':
            return '{0}{1}'.format(self.api_base, url)
        return url

    def _get_client(self):
        token = self.user.get_highest_priority_token()
        if token:
            token = oauth.Token(token, self.user.token_secret)
            client = oauth.Client(self.consumer, token=token)
        else:
            client = oauth.Client(self.consumer)

        return client

    def _prepare_request_body(self, method, url, data):
        if method not in ['POST', 'GET', 'PATCH'] or len(data.keys()) == 0:
            return ''
        if method in ['POST', 'GET']:
            # WARNING: non-primative items in data must be json serialized.
            for key in data:
                if type(data[key]) in [dict, list]:
                    data[key] = json.dumps(data[key])
            return urlencode(data)
        if method == 'PATCH':
            return json.dumps(data)
