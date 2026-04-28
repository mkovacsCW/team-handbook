# Models

A guide to writing clean, maintainable, and performant Django models.

## Model Structure

Follow a consistent internal structure for every model:

```python title="Recommended model layout"
class MyModel(models.Model):
    # 1. Constants
    STATUS_DRAFT = 'draft'
    STATUS_PUBLISHED = 'published'
    STATUS_CHOICES = [
        (STATUS_DRAFT, _('Draft')),
        (STATUS_PUBLISHED, _('Published')),
    ]

    # 2. Database fields
    title = models.CharField(_('title'), max_length=255)
    status = models.CharField(
        _('status'),
        max_length=20,
        choices=STATUS_CHOICES,
        default=STATUS_DRAFT,
    )
    created_at = models.DateTimeField(_('created at'), auto_now_add=True)

    # 3. Foreign keys and relationships
    author = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        related_name='articles',
        verbose_name=_('author'),
    )

    # 4. Custom manager
    objects = MyModelManager()

    # 5. Meta class
    class Meta:
        verbose_name = _('my model')
        verbose_name_plural = _('my models')
        ordering = ['-created_at']

    # 6. __str__ method
    def __str__(self):
        return self.title

    # 7. save method (if needed)
    def save(self, *args, **kwargs):
        super().save(*args, **kwargs)

    # 8. clean method (if needed)
    def clean(self):
        pass

    # 9. Other methods
    def publish(self):
        self.status = self.STATUS_PUBLISHED
        self.save(update_fields=['status'])

    # 10. Properties
    @property
    def is_published(self):
        return self.status == self.STATUS_PUBLISHED
```

## Field Best Practices

### Use `help_text` for admin clarity

```python
title = models.CharField(
    max_length=255,
    unique=True,
    help_text=_('title field'),
)
```

### Be explicit with `null` and `blank`

```python
# String fields: use blank=True, avoid null=True
description = models.TextField(blank=True, default='')

# Non-string fields: null=True is appropriate
published_at = models.DateTimeField(null=True, blank=True)

# Required fields: neither null nor blank
title = models.CharField(max_length=255)
```

### Foreign key best practices

```python
# Always specify on_delete explicitly
author = models.ForeignKey(
    settings.AUTH_USER_MODEL,
    on_delete=models.CASCADE,  # or PROTECT, SET_NULL, etc.
    related_name='articles',
    related_query_name='article',
)
```

!!! tip "Use `settings.AUTH_USER_MODEL`"
    Don't import `User` directly. Using `settings.AUTH_USER_MODEL` keeps your models portable across projects with custom user models.

## Model Inheritance

### Abstract base classes

Use for sharing fields and methods without creating a database table.

```python
class TimeStampedModel(models.Model):
    """Abstract base class with created/updated timestamps."""
    created_at = models.DateTimeField(_('created at'), auto_now_add=True)
    updated_at = models.DateTimeField(_('updated at'), auto_now=True)

    class Meta:
        abstract = True


class Article(TimeStampedModel):
    title = models.CharField(_('title'), max_length=255)
    # Inherits: created_at, updated_at
```

### Multi-table inheritance

Use when you need queryable parent models. Creates a JOIN.

```python
class Place(models.Model):
    name = models.CharField(max_length=100)
    address = models.TextField()


class Restaurant(Place):
    serves_pizza = models.BooleanField(default=False)
    # restaurant.name, restaurant.address work automatically
```

!!! warning "Use sparingly"
    Multi-table inheritance creates implicit JOINs on every query. Prefer abstract base classes unless you genuinely need to query the parent table on its own.

## Mixins

Create reusable functionality with abstract model mixins.

```python
class PublishableMixin(models.Model):
    """Mixin for models that can be published."""
    published = models.BooleanField(_('published'), default=False)
    published_at = models.DateTimeField(_('published at'), null=True, blank=True)

    class Meta:
        abstract = True

    def publish(self):
        self.published = True
        self.published_at = timezone.now()
        self.save(update_fields=['published', 'published_at'])


class SoftDeleteMixin(models.Model):
    """Mixin for soft-deletable models."""
    is_deleted = models.BooleanField(_('deleted'), default=False)
    deleted_at = models.DateTimeField(_('deleted at'), null=True, blank=True)

    class Meta:
        abstract = True

    def soft_delete(self):
        self.is_deleted = True
        self.deleted_at = timezone.now()
        self.save(update_fields=['is_deleted', 'deleted_at'])


# Usage — compose multiple mixins
class Article(TimeStampedModel, PublishableMixin, SoftDeleteMixin):
    title = models.CharField(_('title'), max_length=255)
```

## Meta Class Options

```python
class Article(models.Model):
    # ... fields ...

    class Meta:
        verbose_name = _('article')
        verbose_name_plural = _('articles')

        # Default ordering — use sparingly, impacts all queries
        ordering = ['-created_at', 'title']

        # Unique constraints
        constraints = [
            models.UniqueConstraint(
                fields=['author', 'slug'],
                name='unique_author_slug',
            ),
            models.CheckConstraint(
                condition=models.Q(word_count__gte=0),
                name='positive_word_count',
            ),
        ]

        # Indexes for query optimization
        indexes = [
            models.Index(fields=['status', 'published_at']),
            models.Index(fields=['author', '-created_at']),
        ]

        # Custom permissions
        permissions = [
            ('can_publish', 'Can publish articles'),
        ]
```

## Model Methods

### `__str__`

Always define a meaningful `__str__`:

```python
def __str__(self):
    return self.title
```

!!! warning "Watch for extra queries"
    `f"{self.title} by {self.author.name}"` causes an additional query every time the object is printed (in admin, shell, debug logs). Stick to local fields unless the related object is already loaded.

### `save()`

```python
def save(self, *args, **kwargs):
    # Pre-save logic
    self.slug = self.slug or slugify(self.title)

    # Call the parent save
    super().save(*args, **kwargs)

    # Post-save logic (object now has an ID)
    self.update_search_index()
```

!!! tip "Use `update_fields` when saving specific fields"
    ```python
    def publish(self):
        self.status = self.STATUS_PUBLISHED
        self.published_at = timezone.now()
        self.save(update_fields=['status', 'published_at'])
    ```
    This generates a narrower `UPDATE` and avoids triggering unrelated `pre_save` / `post_save` side effects.

### `clean()`

Used for model-level validation that runs before saving:

```python
from django.core.exceptions import ValidationError

def clean(self):
    if self.start_date and self.end_date:
        if self.start_date > self.end_date:
            raise ValidationError({
                'end_date': _('End date must be after start date.')
            })
```

!!! danger "`clean()` is not called by `save()`"
    You need to call `full_clean()` explicitly, or rely on a form/serializer to invoke it. This trips people up — if your validation isn't running, this is probably why.

### `get_absolute_url()`

Standard Django convention for models with detail views:

```python
from django.urls import reverse

def get_absolute_url(self):
    return reverse('article-detail', kwargs={'slug': self.slug})
```

### Properties

Use `@property` for computed attributes:

```python
@property
def is_published(self):
    return self.status == self.STATUS_PUBLISHED

@property
def reading_time(self):
    words_per_minute = 200
    return max(1, self.word_count // words_per_minute)
```
