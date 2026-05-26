
## Basic Model Structure

```python
from django.db import models
from django.utils import timezone

class Post(models.Model):
    # Basic fields
    title = models.CharField(max_length=200)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    is_published = models.BooleanField(default=False)
    
    def __str__(self):
        return self.title
    
    class Meta:
        ordering = ['-created_at']
```

## Common Field Types

```python
from django.db import models

class Product(models.Model):
    # String fields
    name = models.CharField(max_length=100)
    description = models.TextField(blank=True)
    sku = models.CharField(max_length=50, unique=True)
    slug = models.SlugField(max_length=100, unique=True)
    
    # Numeric fields
    price = models.DecimalField(max_digits=10, decimal_places=2)
    quantity = models.IntegerField(default=0)
    rating = models.FloatField(default=0.0)
    
    # Date/Time fields
    created_date = models.DateField(auto_now_add=True)
    published_time = models.DateTimeField(default=timezone.now)
    
    # File fields
    image = models.ImageField(upload_to='products/', blank=True, null=True)
    document = models.FileField(upload_to='documents/')
    
    # Choice fields
    STATUS_CHOICES = [
        ('draft', 'Draft'),
        ('published', 'Published'),
        ('archived', 'Archived'),
    ]
    status = models.CharField(
        max_length=10,
        choices=STATUS_CHOICES,
        default='draft'
    )
    
    # Other fields
    email = models.EmailField()
    website = models.URLField(blank=True)
    is_active = models.BooleanField(default=True)
```

## Relationships

### One-to-Many (ForeignKey)

```python
class Author(models.Model):
    name = models.CharField(max_length=100)
    bio = models.TextField()

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(
        Author,
        on_delete=models.CASCADE,  # Delete books when author is deleted
        related_name='books',      # Access via author.books.all()
    )
    published_date = models.DateField()
```

### Many-to-Many

```python
class Student(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField()

class Course(models.Model):
    title = models.CharField(max_length=200)
    code = models.CharField(max_length=10)
    students = models.ManyToManyField(
        Student,
        related_name='courses',
        through='Enrollment'  # Custom through table
    )

# Custom through table for extra fields
class Enrollment(models.Model):
    student = models.ForeignKey(Student, on_delete=models.CASCADE)
    course = models.ForeignKey(Course, on_delete=models.CASCADE)
    enrolled_date = models.DateField(auto_now_add=True)
    grade = models.CharField(max_length=2, blank=True, null=True)
    
    class Meta:
        unique_together = ['student', 'course']
```

### One-to-One

```python
class UserProfile(models.Model):
    user = models.OneToOneField(
        'auth.User',
        on_delete=models.CASCADE,
        related_name='profile'
    )
    avatar = models.ImageField(upload_to='avatars/')
    phone = models.CharField(max_length=20)
    address = models.TextField()
```

## Meta Options

```python
class Article(models.Model):
    title = models.CharField(max_length=200)
    
    class Meta:
        # Database table name
        db_table = 'articles'
        
        # Ordering
        ordering = ['-published_date', 'title']
        
        # Unique constraints
        unique_together = ['title', 'author']
        
        # Indexes
        indexes = [
            models.Index(fields=['status', 'published_date']),
        ]
        
        # Verbose names
        verbose_name = 'Article'
        verbose_name_plural = 'Articles'
        
        # Permissions
        permissions = [
            ('can_publish', 'Can publish articles'),
        ]
```

## Custom Methods and Properties

```python
class Order(models.Model):
    customer = models.ForeignKey('Customer', on_delete=models.CASCADE)
    items = models.ManyToManyField('Product', through='OrderItem')
    created_at = models.DateTimeField(auto_now_add=True)
    STATUS_CHOICES = [
        ('pending', 'Pending'),
        ('processing', 'Processing'),
        ('shipped', 'Shipped'),
        ('delivered', 'Delivered'),
    ]
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='pending')
    
    @property
    def total_amount(self):
        """Calculate total order amount"""
        return sum(item.subtotal for item in self.orderitem_set.all())
    
    @property
    def is_completed(self):
        return self.status == 'delivered'
    
    def mark_as_shipped(self):
        """Method to update order status"""
        self.status = 'shipped'
        self.save()
    
    def __str__(self):
        return f"Order #{self.id} - {self.customer.name}"

class OrderItem(models.Model):
    order = models.ForeignKey(Order, on_delete=models.CASCADE)
    product = models.ForeignKey('Product', on_delete=models.CASCADE)
    quantity = models.PositiveIntegerField(default=1)
    unit_price = models.DecimalField(max_digits=10, decimal_places=2)
    
    @property
    def subtotal(self):
        return self.quantity * self.unit_price
```

## Model Inheritance

### Abstract Base Class

```python
class TimeStampedModel(models.Model):
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        abstract = True  # No database table created

class Post(TimeStampedModel):
    title = models.CharField(max_length=200)
    content = models.TextField()
    
class Comment(TimeStampedModel):
    post = models.ForeignKey(Post, on_delete=models.CASCADE)
    text = models.TextField()
```

### Multi-table Inheritance

```python
class Media(models.Model):
    title = models.CharField(max_length=200)
    created_at = models.DateTimeField(auto_now_add=True)

class Video(Media):
    duration = models.IntegerField()  # in seconds
    resolution = models.CharField(max_length=20)

class Photo(Media):
    image = models.ImageField(upload_to='photos/')
    location = models.CharField(max_length=200, blank=True)
```

## Model Managers

```python
class PublishedManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(status='published')
    
    def recent(self):
        return self.get_queryset().order_by('-published_date')[:5]

class Article(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    status = models.CharField(max_length=20, default='draft')
    published_date = models.DateTimeField(null=True, blank=True)
    
    # Custom managers
    objects = models.Manager()  # Default manager
    published = PublishedManager()  # Custom manager
    
    class Meta:
        default_manager_name = 'objects'  # Explicitly set default
```

