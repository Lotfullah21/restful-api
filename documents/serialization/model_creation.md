# Model creation using DRF

ModelSerializer, handles creation of objects and by default serializers are the ones that save the object to the database.

## 1. create Method in Serializers

The create method in a serializer is used to encapsulate the logic for creating a model instance. It focuses on data validation and model creation.

#### Use Case:

- When we want the serializer to handle the logic for saving data.
- Useful for keeping views lightweight, focusing only on request/response logic.
- Ideal for simple object creation where only model data is involved.

#### Advantages:

- Keeps the creation logic close to where data validation happens.
- Promotes reusability: we can use the same serializer in multiple views or contexts (e.g., APIs, forms).

##### When we send a POST request to an API using this serializer:

1. The serializer validates the input data.
2. The default create method is called after validation, which uses the `save()` method to create a new instance of the model.

##### Default Behavior

The default create method of ModelSerializer:

Takes the validated_data (which is already validated by the serializer).
Creates a new model instance using the model.objects.create(\*\*validated_data) pattern.

## Custom create method

#### When Is a Custom create Method Necessary?

we need a custom create method when:

we have nested relationships or custom field handling that the default implementation cannot process.
For example, `category_id` here is write-only and doesn't directly correspond to a model field (since it's a ForeignKey relationship). The serializer needs to process it manually.

```py
from rest_framework import serializers
from .models import Course, CourseCategory
class CategorySerializer(serializers.ModelSerializer):
    class Meta:
        model = CourseCategory
        fields=["id","category_name"]
class CourseSerializer(serializers.ModelSerializer):
    category_id = serializers.PrimaryKeyRelatedField(queryset = CourseCategory.objects.all(), write_only=True)
    category = CategorySerializer(source="category_id", read_only=True)
    class Meta:
        model = Course
        fields = ["id","course_name","slug","category_id","category"]
```

## 2. create Method in Views

The create method in a view handles the `request-response` cycle and often uses serializers to perform the actual data creation.

#### Use Case:

- When we need custom logic before or after saving the data (e.g., logging, notifications).
- When the creation process requires complex interactions with other services or models.
- When we want to return customized responses instead of just serialized data.

#### Advantages:

- Allows us to include additional business logic during creation.
- Gives more control over the response format.
- we can perform pre-save or post-save actions in a single place.

#### Example usage

```py

class CourseList(generics.ListCreateAPIView):
    queryset = Course.objects.all()
    serializer_class = CourseSerializer

    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        # Saves the data using the serializer's create method
        self.perform_create(serializer)
        response_data = {
            "message": "Course created successfully!",
            "course": serializer.data,
        }
        return Response(response_data, status=status.HTTP_201_CREATED)

```

## When to Use Which?

#### Use create in the serializer:

- When the logic is focused on creating a single model instance.
- To keep the creation logic reusable and separate from view concerns.
- If the creation process is simple and doesn't involve additional processing.

#### Use create in the view:

- When you need to handle custom actions around creation (e.g., sending an email, triggering an event).
- When you want to customize the response or include additional information.
- If the creation process involves external services or complex workflows.

#### self.perform_create(serializer)

```py
class CourseList(generics.ListCreateAPIView):
    queryset = Course.objects.all()
    serializer_class = CourseSerializer

    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data = request.data)
        serializer.is_valid(raise_exception=True)
        self.perform_create(serializer)
        response_data = {
            "message":"Course Created successfully!!!!",
            "course":serializer.data

        }
        return Response(response_data, status.HTTP_201_CREATED)

```

| **Aspect**          | **`serializer.save()`**                           | **`self.perform_create(serializer)`**                          |
| ------------------- | ------------------------------------------------- | -------------------------------------------------------------- |
| **Direct/Indirect** | Directly saves the instance.                      | A wrapper for `serializer.save()` with room for customization. |
| **Scope of Use**    | Used anywhere in views or serializers.            | Used in generic views to add logic before or after saving.     |
| **Customization**   | No built-in hook for additional logic.            | Designed for additional logic (e.g., logging, notifications).  |
| **Default Usage**   | Core to saving the serializer data.               | Used in generic views like `CreateAPIView`.                    |
| **Purpose**         | Saves the object by calling `create` or `update`. | Adds custom logic before or after calling `serializer.save()`. |
| **Typical Context** | Function-based views or custom logic handling.    | Generic views like `CreateAPIView` or `ListCreateAPIView`.     |

#### When to Use

Use `serializer.save()`:

If you're manually handling serialization and saving, especially in function-based views.
When you don't need any additional pre/post-save logic.

Use `self.perform_create(serializer)`:

When working with generic views, and you want to include custom logic while maintaining clean and modular code.
