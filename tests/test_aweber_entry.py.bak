import re
from unittest import TestCase
from urllib import urlencode

from aweber_api import AWeberAPI, AWeberCollection, AWeberEntry
from aweber_api.base import APIException
from mock_adapter import MockAdapter


class TestAWeberEntry(TestCase):

    def setUp(self):
        self.aweber = AWeberAPI('1', '2')
        self.aweber.adapter = MockAdapter()
        self.list = self.aweber.load_from_url('/accounts/1/lists/303449')

    def test_should_be_an_entry(self):
        self.assertEqual(type(self.list), AWeberEntry)
        self.assertEqual(self.list.type, 'list')

    def test_should_have_id(self):
        self.assertEqual(self.list.id, 303449)

    def test_should_have_other_properties(self):
        self.assertEqual(self.list.name, 'default303449')

    def test_should_have_child_collections(self):
        campaigns = self.list.campaigns
        self.assertEqual(type(campaigns), AWeberCollection)

    def test_findSubscribers_should_handle_errors(self):
        account = self.aweber.load_from_url('/accounts/1')
        self.assertRaises(APIException, account.findSubscribers, name='bob')


class AccountTestCase(TestCase):

    def setUp(self):
        self.aweber = AWeberAPI('1', '2')
        self.aweber.adapter = MockAdapter()
        self.account = self.aweber.load_from_url('/accounts/1')


class ListTestCase(TestCase):

    def setUp(self):
        self.aweber = AWeberAPI('1', '2')
        self.aweber.adapter = MockAdapter()
        self.list_ = self.aweber.load_from_url('/accounts/1/lists/303449')


class TestAWeberAccountEntry(AccountTestCase):

    def test_should_be_an_entry(self):
        self.assertEqual(type(self.account), AWeberEntry)
        self.assertEqual(self.account.type, 'account')


class TestAccountGetWebForms(AccountTestCase):

    def setUp(self):
        super(TestAccountGetWebForms, self).setUp()
        self.forms = self.account.get_web_forms()

    def test_should_be_a_list(self):
        self.assertEqual(type(self.forms), list)

    def test_should_have_181_web_forms(self):
        self.assertEqual(len(self.forms), 181)

    def test_each_should_be_entry(self):
        for entry in self.forms:
            self.assertEqual(type(entry), AWeberEntry)
            self.assertEqual(entry.type, 'web_form')

    def test_each_should_have_correct_url(self):
        url_regex = '/accounts\/1\/lists\/\d*/web_forms/\d*'
        for entry in self.forms:
            self.assertTrue(re.match(url_regex, entry.url))


class TestAccountGetWebFormSplitTests(AccountTestCase):

    def setUp(self):
        super(TestAccountGetWebFormSplitTests, self).setUp()
        self.forms = self.account.get_web_form_split_tests()

    def test_should_be_a_list(self):
        self.assertEqual(type(self.forms), list)

    def test_should_have_10_split_tests(self):
        self.assertEqual(len(self.forms), 10)

    def test_each_should_be_entry(self):
        for entry in self.forms:
            self.assertEqual(type(entry), AWeberEntry)
            self.assertEqual(entry.type, 'web_form_split_test')

    def test_each_should_have_correct_url(self):
        url_regex = '/accounts\/1\/lists\/\d*/web_form_split_tests/\d*'
        for entry in self.forms:
            self.assertTrue(re.match(url_regex, entry.url))


class TestAccountFindSubscribers(AccountTestCase):

    def test_should_support_find_method(self):
        base_url = '/accounts/1'
        account = self.aweber.load_from_url(base_url)
        self.aweber.adapter.requests = []
        subscribers = account.findSubscribers(email='joe@example.com')
        request = self.aweber.adapter.requests[0]

        assert subscribers != False
        assert isinstance(subscribers, AWeberCollection)
        assert len(subscribers) == 1
        assert subscribers[0].self_link == \
                'https://api.aweber.com/1.0/accounts/1/lists/303449/subscribers/1'


class TestListScheduleBroadcast(ListTestCase):

    def setUp(self):
        super(TestListScheduleBroadcast, self).setUp()
        self.aweber.adapter.requests = []
        self.status = self.list_.schedule_broadcast(
            bc_id=2, scheduled_for='2014-09-06 18:55:00')
        self.request = self.aweber.adapter.requests[0]

    def test_should_return_status(self):
        self.assertEqual(int(self.status), 201)

    def test_should_make_post_request(self):
        self.assertEqual(self.request['method'], 'POST')

    def test_should_build_correct_url(self):
            self.assertEqual(self.request['url'],
            '/accounts/1/lists/303449/broadcasts/2/schedule'
        )

    def test_should_pass_scheduled_for_date(self):
            self.assertEqual(self.request['data'],
            {'scheduled_for': '2014-09-06 18:55:00'}
        )


class TestListScheduleBroadcastError(ListTestCase):

    def setUp(self):
        super(TestListScheduleBroadcastError, self).setUp()
        self.list_ = self.aweber.load_from_url('/accounts/1/lists/303449')
        self.aweber.adapter.requests = []

    def test_should_raise_exception_when_failing(self):
        self.assertRaises(
            APIException,
            self.list_.schedule_broadcast,
            bc_id=3,
            scheduled_for='2014-09-06 18:55:00',
        )


class TestListCancelBroadcast(ListTestCase):

    def setUp(self):
        super(TestListCancelBroadcast, self).setUp()
        self.aweber.adapter.requests = []
        self.status = self.list_.cancel_broadcast(bc_id=2)
        self.request = self.aweber.adapter.requests[0]

    def test_should_return_status(self):
        self.assertEqual(int(self.status), 204)

    def test_should_make_post_request(self):
        self.assertEqual(self.request['method'], 'POST')

    def test_should_build_correct_url(self):
            self.assertEqual(self.request['url'],
            '/accounts/1/lists/303449/broadcasts/2/cancel'
        )

    def test_should_pass_empty_date(self):
            self.assertEqual(self.request['data'], {})


class TestListCancelBroadcastError(ListTestCase):

    def setUp(self):
        super(TestListCancelBroadcastError, self).setUp()
        self.list_ = self.aweber.load_from_url('/accounts/1/lists/303449')
        self.aweber.adapter.requests = []

    def test_should_raise_exception_when_failing(self):
        self.assertRaises(
            APIException,
            self.list_.cancel_broadcast,
            bc_id=3,
        )


class TestListGetBroadcasts(ListTestCase):

    def setUp(self):
        super(TestListGetBroadcasts, self).setUp()
        self.aweber.adapter.requests = []
        self.broadcasts = self.list_.get_broadcasts(status='sent')
        self.request = self.aweber.adapter.requests[0]

    def test_should_return_collection(self):
        self.assertEqual(type(self.broadcasts), AWeberCollection)

    def test_should_make_get_request(self):
        self.assertEqual(self.request['method'], 'GET')

    def test_should_build_correct_url(self):
        self.assertEqual(
            self.request['url'],
            '/accounts/1/lists/303449/broadcasts?status=sent'
        )


class SubscriberTestCase(TestCase):

    def setUp(self):
        self.aweber = AWeberAPI('1', '2')
        self.aweber.adapter = MockAdapter()
        sub_url = '/accounts/1/lists/303449/subscribers/1'
        self.subscriber = self.aweber.load_from_url(sub_url)


class TestGetAndSetData(SubscriberTestCase):

    def test_get_name(self):
        self.assertEqual(self.subscriber.name, 'Joe Jones')

    def test_set_name(self):
        self.subscriber.name = 'Randy Rhodes'
        self.assertEqual(self.subscriber.name, 'Randy Rhodes')

    def test_get_custom_fields(self):
        fields = self.subscriber.custom_fields
        self.assertEqual(fields['Color'], 'blue')

    def test_set_custom_fields(self):
        self.subscriber.custom_fields['Color'] = 'Red'
        self.assertEqual(self.subscriber._data['custom_fields']['Color'], 'Red')
        fields = self.subscriber.custom_fields
        self.assertEqual(fields['Color'], 'Red')

    def test_should_be_able_get_activity(self):
        activity = self.subscriber.get_activity()


class TestMovingSubscribers(TestCase):

    def setUp(self):
        self.aweber = AWeberAPI('1', '2')
        self.aweber.adapter = MockAdapter()
        subscriber_url = '/accounts/1/lists/303449/subscribers/1'
        new_list_url = '/accounts/1/lists/505454'
        self.subscriber = self.aweber.load_from_url(subscriber_url)
        self.subscriber._diff['name'] = 'Joe Schmoe'
        self.list = self.aweber.load_from_url(new_list_url)
        self.move_subscriber()

    def move_subscriber(self, **kwargs):
        self.aweber.adapter.requests = []
        self.resp = self.subscriber.move(self.list, **kwargs)
        self.move_req = self.aweber.adapter.requests[0]
        self.get_req = self.aweber.adapter.requests[1]

    def test_returned_true(self):
        self.assertTrue(self.resp)

    def test_should_have_requested_move_with_post(self):
        self.assertEqual(self.move_req['method'], 'POST')

    def test_should_have_requested_move_on_subscriber(self):
        self.assertEqual(self.move_req['url'] , self.subscriber.url)

    def test_should_have_requested_move_with_correct_parameters(self):
        expected_params = {'ws.op': 'move', 'list_link': self.list.self_link}
        self.assertEqual(self.move_req['data'], expected_params)

    def test_should_make_two_requests(self):
        self.assertEqual(len(self.aweber.adapter.requests), 2)

    def test_should_refresh_subscriber_resource(self):
        self.assertEqual(self.get_req['method'], 'GET')
        self.assertEqual(self.get_req['url'] ,
            '/accounts/1/lists/505454/subscribers/3')

    def test_should_reset_diff(self):
        self.assertEqual(self.subscriber._diff, {})

    def test_should_accept_last_followup_message_number_sent(self):
        self.move_subscriber(last_followup_message_number_sent=999)
        expected_params = {'ws.op': 'move', 'list_link': self.list.self_link,
                           'last_followup_message_number_sent': 999}

        self.assertEqual(self.move_req['data'], expected_params)

class TestSavingSubscriberData(SubscriberTestCase):

    def setUp(self):
        super(TestSavingSubscriberData, self).setUp()
        self.aweber.adapter.requests = []
        self.subscriber.name = 'Gary Oldman'
        self.subscriber.custom_fields['Color'] = 'Red'
        self.resp = self.subscriber.save()
        self.req = self.aweber.adapter.requests[0]

    def test_returned_true(self):
        self.assertTrue(self.resp)

    def test_should_make_request(self):
        self.assertEqual(len(self.aweber.adapter.requests), 1)

    def test_should_have_requested_resource_url(self):
        self.assertEqual(self.req['url'] , self.subscriber.url)

    def test_should_have_requested_with_patch(self):
        self.assertEqual(self.req['method'], 'PATCH')

    def test_should_have_supplied_data(self):
        self.assertEqual(self.req['data']['name'], 'Gary Oldman')

    def test_should_not_include_unchanged_data(self):
        self.assertFalse('email' in self.req['data'])

    def test_should_given_all_custom_fields(self):
        # Make changed, Model did not
        self.assertEqual(self.req['data']['custom_fields']['Color'], 'Red')
        self.assertEqual(self.req['data']['custom_fields']['Walruses'], '')


class TestSavingInvalidSubscriberData(TestCase):

    def setUp(self):
        self.aweber = AWeberAPI('1', '2')
        self.aweber.adapter = MockAdapter()
        sub_url = '/accounts/1/lists/303449/subscribers/2'
        self.subscriber = self.aweber.load_from_url(sub_url)
        self.subscriber.name = 'Gary Oldman'
        self.subscriber.custom_fields['New Custom Field'] = 'Cookies'

    def test_save_failed(self):
        self.assertRaises(APIException, self.subscriber.save)


class TestDeletingSubscriberData(SubscriberTestCase):

    def setUp(self):
        super(TestDeletingSubscriberData, self).setUp()
        self.aweber.adapter.requests = []
        self.response = self.subscriber.delete()
        self.req = self.aweber.adapter.requests[0]

    def test_should_be_deleted(self):
        self.assertTrue(self.response)

    def test_should_have_made_request(self):
        self.assertEqual(len(self.aweber.adapter.requests), 1)

    def test_should_have_made_delete(self):
        self.assertEqual(self.req['method'], 'DELETE')


class TestFailedSubscriberDelete(TestCase):

    def setUp(self):
        self.aweber = AWeberAPI('1', '2')
        self.aweber.adapter = MockAdapter()
        sub_url = '/accounts/1/lists/303449/subscribers/2'
        self.subscriber = self.aweber.load_from_url(sub_url)

    def test_should_raise_exception_when_failing(self):
        self.assertRaises(APIException, self.subscriber.delete)


class TestGettingParentEntry(TestCase):

    def setUp(self):
        self.aweber = AWeberAPI('1', '2')
        self.aweber.adapter = MockAdapter()
        self.list = self.aweber.load_from_url('/accounts/1/lists/303449')
        self.account = self.aweber.load_from_url('/accounts/1')
        self.custom_field = self.aweber.load_from_url('/accounts/1/lists/303449/custom_fields/1')

    def test_should_be_able_get_parent_entry(self):
        entry = self.list.get_parent_entry()

    def test_list_parent_should_be_account(self):
        entry = self.list.get_parent_entry()
        self.assertEqual(type(entry), AWeberEntry)
        self.assertEqual(entry.type, 'account')

    def test_custom_field_parent_should_be_list(self):
        entry = self.custom_field.get_parent_entry()
        self.assertEqual(type(entry), AWeberEntry)
        self.assertEqual(entry.type, 'list')

    def test_account_parent_should_be_none(self):
        entry = self.account.get_parent_entry()
        self.assertEqual(entry, None)
