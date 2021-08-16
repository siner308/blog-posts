---
layout: post
title:  "장고의 패스워드 유효성 검사 코드분석"
subtitle: "Django Password Validator"
author: "Siner"
catalog: true
header-mask:  0.3
tags:
    - Django
    - Auth
    - Validator
    - 
date:   2021-08-16
multilingual: false
---

django의 [startproject](https://docs.djangoproject.com/en/3.2/ref/django-admin/#django-admin-startproject)로 기본적으로 생성되는 `AUTH_PASSWORD_VALIDATORS`에 대해 분석해보려 합니다

```python
# Password validation
# https://docs.djangoproject.com/en/3.2/ref/settings/#auth-password-validators
AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]
```

## 1. UserAttributeSimilarityValidator

유저의 attributes(username, first_nae, last_name, email)를 정규표현식으로 split한 후 각각의 value_part에 대해 유사도를 측정하고, 유사도가 max_similarity(default: 0.7)이상인 경우 validation error를 발생시킵니다. 유사도 측정에는 [SequenceMatcher](https://docs.python.org/ko/3/library/difflib.html#difflib.SequenceMatcher.quick_ratio)를 사용합니다.

<details>
<summary style='color: blue'>코드보기</summary>

```python
class UserAttributeSimilarityValidator:
    """
    Validate whether the password is sufficiently different from the user's
    attributes.

    If no specific attributes are provided, look at a sensible list of
    defaults. Attributes that don't exist are ignored. Comparison is made to
    not only the full attribute value, but also its components, so that, for
    example, a password is validated against either part of an email address,
    as well as the full address.
    """
    DEFAULT_USER_ATTRIBUTES = ('username', 'first_name', 'last_name', 'email')

    def __init__(self, user_attributes=DEFAULT_USER_ATTRIBUTES, max_similarity=0.7):
        self.user_attributes = user_attributes
        self.max_similarity = max_similarity

    def validate(self, password, user=None):
        if not user:
            return

        for attribute_name in self.user_attributes:
            value = getattr(user, attribute_name, None)
            if not value or not isinstance(value, str):
                continue
            value_parts = re.split(r'\W+', value) + [value]
            for value_part in value_parts:
                if SequenceMatcher(a=password.lower(), b=value_part.lower()).quick_ratio() >= self.max_similarity:
                    try:
                        verbose_name = str(user._meta.get_field(attribute_name).verbose_name)
                    except FieldDoesNotExist:
                        verbose_name = attribute_name
                    raise ValidationError(
                        _("The password is too similar to the %(verbose_name)s."),
                        code='password_too_similar',
                        params={'verbose_name': verbose_name},
                    )

    def get_help_text(self):
        return _('Your password can’t be too similar to your other personal information.')
```
</details>

## 2. MinimumLengthValidator

password의 길이가 min_length(default: 8)보다 작다면 validatoion error를 발생시킵니다.

<details>
<summary style='color: blue'>코드보기</summary>

```python
class MinimumLengthValidator:
    """
    Validate whether the password is of a minimum length.
    """
    def __init__(self, min_length=8):
        self.min_length = min_length

    def validate(self, password, user=None):
        if len(password) < self.min_length:
            raise ValidationError(
                ngettext(
                    "This password is too short. It must contain at least %(min_length)d character.",
                    "This password is too short. It must contain at least %(min_length)d characters.",
                    self.min_length
                ),
                code='password_too_short',
                params={'min_length': self.min_length},
            )

    def get_help_text(self):
        return ngettext(
            "Your password must contain at least %(min_length)d character.",
            "Your password must contain at least %(min_length)d characters.",
            self.min_length
        ) % {'min_length': self.min_length}
```
</details>

## 3. CommonPasswordValidator

[사람들이 가장 많이 사용하는 패스워드 20000개](https://gist.github.com/roycewilliams/281ce539915a947a23db17137d91aeb7)에 해당하는 경우 validator error를 발생시킵니다. 2000개의 패스워드는 gzip으로 압축되어 django프로젝트에 내장되어있습니다. (django/contrib/auth/common-passwords.txt.gz)

<details>
<summary style='color: blue'>코드보기</summary>

```python
class CommonPasswordValidator:
    """
    Validate whether the password is a common password.

    The password is rejected if it occurs in a provided list of passwords,
    which may be gzipped. The list Django ships with contains 20000 common
    passwords (lowercased and deduplicated), created by Royce Williams:
    https://gist.github.com/roycewilliams/281ce539915a947a23db17137d91aeb7
    The password list must be lowercased to match the comparison in validate().
    """
    DEFAULT_PASSWORD_LIST_PATH = Path(__file__).resolve().parent / 'common-passwords.txt.gz'

    def __init__(self, password_list_path=DEFAULT_PASSWORD_LIST_PATH):
        try:
            with gzip.open(password_list_path, 'rt', encoding='utf-8') as f:
                self.passwords = {x.strip() for x in f}
        except OSError:
            with open(password_list_path) as f:
                self.passwords = {x.strip() for x in f}

    def validate(self, password, user=None):
        if password.lower().strip() in self.passwords:
            raise ValidationError(
                _("This password is too common."),
                code='password_too_common',
            )

    def get_help_text(self):
        return _('Your password can’t be a commonly used password.')
```
</details>

## 4. NumericPasswordValidator

password가 숫자로만 이루어진 경우 validation error를 발생시킵니다.

<details>
<summary style='color: blue'>코드보기</summary>

```python
class NumericPasswordValidator:
    """
    Validate whether the password is alphanumeric.
    """
    def validate(self, password, user=None):
        if password.isdigit():
            raise ValidationError(
                _("This password is entirely numeric."),
                code='password_entirely_numeric',
            )

    def get_help_text(self):
        return _('Your password can’t be entirely numeric.')
```
</details>
