# Hipo DRF Exceptions
[![hipo](https://img.shields.io/badge/hipo-red.svg)](https://hipolabs.com) [![Build Status](https://travis-ci.org/Hipo/hipo-drf-exceptions.svg?branch=master)](https://travis-ci.org/Hipo/hipo-drf-exceptions) [![pypi](https://img.shields.io/pypi/v/hipo-drf-exceptions.svg)](https://pypi.org/project/hipo-drf-exceptions/)

A [Django](https://www.djangoproject.com) app for returning consistent, verbose and easy to parse error messages on [Django Rest Framework](https://www.django-rest-framework.org/) backends.

Parsing error messages generated by DRF is a bit of pain for client developers. They need to try-catch different possible error formats. When you add errors raised at the business logic level, the error parsing becomes even more difficult. 

This package unifies the output format of DRF in the "Hipo Error Protocol". 

*No more "An error occured." errors.*

This package also provides the "fallback message", a text string that always contains a human readable error summary. This way, client developers can always fallback and show this message when the client receives an error that is not handled.

> Sounds cool! Can client devs just use this field all the time?

In our past experience, we noticed that some _lazy_ client developers tend to use this message and avoid writing any code to parse the error bundle. However, the message in this field is automatically generated and may not always be suitable for end users. In order to make clear that this is a *fallback*  message, we named this field "fallback_message"


## Table of Contents

- [Installation](#installation)
- [Usage](#usage)
- [Support](#support)
- [Contributing](#contributing)

## Installation

You can get stable version of Hipo Excepitons by using pip, pipenv or poetry:
```
pip install hipo-drf-exceptions
```

## Usage

### Handler
You will need to set `EXCEPTION_HANDLER` of the `REST_FRAMEWORK` setting of your Django project settings.py file.
```
REST_FRAMEWORK = {
    ..
    'EXCEPTION_HANDLER': 'hipo_drf_exceptions.handler',
}
```

### Example Error Responses

#### Field Error

You can make validations on model level and raise `ValidationError` when it is required.
```python
from django.core.exceptions import ValidationError

class Invitation(models.Model):
    email = models.EmailField(unique=True)

    def save(self, *args, **kwargs):
        if User.objects.filter(email=self.email).exists():
            raise ValidationError({"email": _("Email is already registered.")})
            
        super().save(*args, **kwargs)
```

If the view or serializer encounters with the `ValidationError`, The response will be like:
```json
{
    "type": "ValidationError",
    "detail": {
        "email": [
            "Email is already registered."
        ]
    },
    "fallback_message": "Email is already registered."
}
```

#### Non Field Error
Implement your own error classes.
```python
from hipo_drf_exceptions import BaseAPIException

class ProfileCredentialError(BaseAPIException):
    default_detail = _('Profile credentials are not correct.')
```

Raise error when it is required.
```python
class AuthenticationView(GenericAPIView):

    def post(self, request, *args, **kwargs):
        ..
        if not profile.check_password(password):
            raise ProfileCredentialError()
        ..
```

The response will be like:
```json
{
    "type": "ProfileCredentialError",
    "detail": {
        "non_field_errors": [
            "Profile credentials are not correct."
        ]
    },
    "fallback_message": "Profile credentials are not correct."
}
```

## Support

Please [open an issue](https://github.com/hipo/hipo-drf-exceptions/issues/new) for support.

## Contributing

Please contribute using [Github Flow](https://guides.github.com/introduction/flow/). Create a branch, add commits, and [open a pull request](https://github.com/hipo/hipo-drf-exceptions/compare/).
