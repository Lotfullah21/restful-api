#### Sanitize HTML and JavaScript

Unless it is intended, we should always check if the user client added an HTML tag inside the data and neutralized it by converting special HTML characters into HTML entities. This is because hackers can use `<script>` tags to inject JavaScript and `<img>` tags to add unwanted trackers.

There is a popular third-party package called `bleach` that can help you to clean this. It will convert all HTML special characters like <’, ‘> and other tags to HTML entities so that the browser doesn’t execute them as HTML anymore.

### Installation

#### Step 1

```sh
pip3 install bleach
```

#### Step 2

Import the bleach module in the serializers.py file.

```sh
import bleach
```

#### Step 3

Sanitize the field data using both the validate_field() and validate() methods. Inside these validation methods, you have to use the clean() function provided by the bleach module to clean up the input data.

```py
    def validate_title(self, value):
        if len(value)<10:
            raise serializers.ValidationError("Price cannot be less than 10")
        return bleach.clean(value)
```

#### !Important

the main intention of bleach is to avoid html insertion in text based input, you cannot use bleach for number type values.

```py
def validate(self, attrs):
        attrs['title'] = bleach.clean(attrs['title'])
        if(attrs['price']<2):
            raise serializers.ValidationError('Price should not be less than 2.0')
        return super().validate(attrs)
```
