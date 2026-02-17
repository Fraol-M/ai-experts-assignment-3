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

The fix adds a defensive check to the refresh condition:

```python
if not self.oauth2_token or not isinstance(self.oauth2_token, OAuth2Token) or (
    isinstance(self.oauth2_token, OAuth2Token) and self.oauth2_token.expired
):
```

This ensures that **any** token value that isn't a valid `OAuth2Token` instance (whether it's a `dict`, `None`, or some other incorrect type) will trigger a refresh. When the token is a dict (as in the bug case), `not isinstance(..., OAuth2Token)` is `True`, so `refresh_oauth2()` is called, replacing the invalid token with a valid one. The subsequent header-setting logic then works correctly.

