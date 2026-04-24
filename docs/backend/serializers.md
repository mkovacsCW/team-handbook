# Serializers

## 1. Use purpose-specific serializers

Create separate serializers for different use cases: list views, detail views, and create/update operations. This keeps payloads lean and avoids over-fetching data.

## 2. Leverage `source` for simple field mappings

Use the `source` argument instead of `SerializerMethodField` when you just need to traverse relationships or rename fields.

```python
# Good — use source
job_number = serializers.CharField(source='job.job_no', read_only=True)

# Avoid when unnecessary
def get_job_number(self, obj):
    return obj.job.job_no if obj.job else None
```

## 3. Be explicit with `read_only=True`

Always mark computed or derived fields as read-only to prevent validation errors on create/update.

## 4. Use `select_related` and `prefetch_related` in your views

Serializers that access related objects can cause N+1 queries. Optimize in your viewset:

```python
class LocateViewSet(viewsets.ModelViewSet):
    def get_queryset(self):
        return Locate.objects.select_related('requested_by', 'job')
```

## 5. Keep `SerializerMethodField` logic simple

If the logic in a method field grows complex, move it to a model property or manager method:

```python title="models.py"
@property
def project_name_display(self):
    if self.job and self.job.name:
        parts = self.job.name.split(' - ', 1)
        return parts[1] if len(parts) > 1 else self.job.name
    return None
```

```python title="serializers.py"
project_name = serializers.CharField(source='project_name_display', read_only=True)
```

## 6. Use `get_user_model()` instead of importing `User` directly

Always use `get_user_model()` for portability across projects with custom user models.

## 7. Validate in the serializer when appropriate

Add custom validation for create/update serializers:

```python
class LocateCreateSerializer(serializers.ModelSerializer):
    class Meta:
        model = Locate
        fields = ['location_name', 'job']

    def validate_job(self, value):
        if value and not value.is_active:
            raise serializers.ValidationError("Cannot create locate for inactive job.")
        return value
```

## 8. Set the user in `perform_create`, not the serializer

Moving user assignment to the view keeps serializers more reusable:

```python title="views.py"
def perform_create(self, serializer):
    serializer.save(requested_by=self.request.user)
```

```python title="serializers.py"
class LocateCreateSerializer(serializers.ModelSerializer):
    class Meta:
        model = Locate
        fields = ['location_name', 'job']
```

## 9. Use `fields` explicitly, avoid `exclude`

Explicit field lists are safer and more maintainable than `exclude`, which can accidentally expose new fields added to models.

## 10. Use `to_representation` for complex transformations

If you need to heavily customize output without adding many method fields:

```python
def to_representation(self, instance):
    data = super().to_representation(instance)
    data['is_urgent'] = instance.business_days_remaining <= 2
    return data
```
