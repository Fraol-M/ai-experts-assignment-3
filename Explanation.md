# Explanation

## What was the bug?

In `app/http_client.py`, the `request()` method failed to refresh the OAuth2 token when `oauth2_token` was set to a **dict** instead of an `OAuth2Token` instance. This meant that API requests made with a dict-typed token would be sent **without any Authorization header**.

## Why did it happen?

The refresh condition on line 30 was:

```python
if not self.oauth2_token or (
    isinstance(self.oauth2_token, OAuth2Token) and self.oauth2_token.expired
):
```

A non-empty dict like `{"access_token": "stale", "expires_at": 0}` is truthy, so `not self.oauth2_token` evaluates to `False`. And since a dict is not an `OAuth2Token` instance, the `isinstance` check also evaluates to `False`. The entire condition is `False`, so `refresh_oauth2()` is never called for dict tokens.

Subsequently, the header-setting block (`if isinstance(self.oauth2_token, OAuth2Token)`) is also skipped, resulting in no `Authorization` header.

## What is the Fix?

The fix simplifies and hardens the refresh condition to:

```python
if not isinstance(self.oauth2_token, OAuth2Token) or self.oauth2_token.expired:
```

This works because of short-circuit evaluation. If the token is `None` or a `dict`, the first part is `True`, so Python does not evaluate `.expired` on an invalid type and refresh is triggered immediately. If the token is a real `OAuth2Token`, the first part is `False`, then `.expired` is checked safely. This guarantees invalid token types are refreshed and valid-but-expired tokens are refreshed too.

