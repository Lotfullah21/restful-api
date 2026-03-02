## Data validation

data validation is an important step and it ensures that user data is valid and sufficient.
Some validations are for instance, the price of an item cannot be less than 0 or title of courses cannot be same.

there are four common ways to validate the data

### 1: Conditions in the field

directly we add our condition before in Meta class in serializers.

```py
 course_price = serializers.FloatField(source = "price",min_value = 10)
```

Now, the min value we can add to `course_price` is 10, if the price value is less that 10, it will raise an error.

### 2. Using keyword arguments in the Meta class

in Meta class, we can add additional validation for every field.

```py
class Meta:
        model = MenuItem
        fields = ['id','title','price','price_after_tax','category','category_id']
        extra_kwargs = {
            'price': {'min_value': 10},
        }
```

### 3. Using validate_field() method

we can use a builtin function `validate_fieldName(self, value)`, where `fieldName` represents the name of each field and `value` represents it is value.

```py
    def validate_price(self, value):
        if value<2:
            raise serializers.ValidationError("Price cannot be less than 10")
```

### 4. Using the validate() method

```py
def validate(self, attrs):
    if attrs['price'] < 10:
        raise serializers.ValidationError('Price should not be less than 10')

    if len(attrs['title']) < 8:
        raise serializers.ValidationError('Title cannot have short names (minimum 8 characters)')
    # call the base class validate method for any default validation
    return super().validate(attrs)
```

## Unique validation

we need to make sure that there is no duplicate entry made by the clients. In such cases, unique validators become useful. Using this validator, we can ensure the uniqueness of a single field or combination of fields.

```py
from rest_framework.validators import UniqueValidator
extra_kwargs = {'title': {'validators': [UniqueValidator(queryset=Course.objects.all())]}}
```
