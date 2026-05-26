
## **All Django Model Field Types**

### **1. AutoField** *(Auto-incrementing integer)*

```python
class Product(models.Model):
    # Implicitly created as 'id' field if not defined
    product_id = models.AutoField(primary_key=True)  # INT AUTO_INCREMENT
    
    # BigAutoField - For larger tables (64-bit)
    big_id = models.BigAutoField(primary_key=True)  # BIGINT AUTO_INCREMENT
    
    # SmallAutoField - For small tables (16-bit, Django 3.0+)
    small_id = models.SmallAutoField(primary_key=True)  # SMALLINT AUTO_INCREMENT
```

### **2. BigIntegerField** *(64-bit integer)*

```python
class Analytics(models.Model):
    views_count = models.BigIntegerField(default=0)  # BIGINT
    # Range: -9223372036854775808 to 9223372036854775807
```

### **3. BinaryField** *(Raw binary data)*

```python
class Document(models.Model):
    file_data = models.BinaryField(
        blank=True,
        null=True,
        editable=True  # Whether field can be edited in forms
    )  # BYTEA or BLOB
    
    hash_value = models.BinaryField(max_length=32)  # For storing hashes
```

### **4. BooleanField** *(True/False)*

```python
class User(models.Model):
    is_active = models.BooleanField(
        default=True,
        help_text="Designates whether this user should be treated as active."
    )  # BOOL or TINYINT(1)
    
    is_staff = models.BooleanField(default=False)  # BOOL
    
    # NullBooleanField (deprecated in Django 2.1+, use BooleanField(null=True) instead)
    # NullBooleanField stores True, False, or None
```

### **5. CharField** *(String with max length)*

```python
class Article(models.Model):
    title = models.CharField(
        max_length=255,           # Required: maximum characters
        db_collation='utf8_bin',  # Database collation (Django 3.2+)
    )  # VARCHAR(255)
    
    # Optimized for database performance with db_index
    slug = models.CharField(
        max_length=100,
        db_index=True,
        unique=True
    )  # VARCHAR(100) with index
```

### **6. DateField** *(Date only, no time)*

```python
class Event(models.Model):
    event_date = models.DateField(
        auto_now=False,      # Automatically set to now on every save
        auto_now_add=False,  # Automatically set to now on creation only
        default='2024-01-01',
        null=True,
        blank=True
    )  # DATE
    
    birth_date = models.DateField()  # DATE
```

### **7. DateTimeField** *(Date and time)*

```python
class Post(models.Model):
    created_at = models.DateTimeField(
        auto_now_add=True  # Set on creation only
    )  # DATETIME or TIMESTAMP
    
    updated_at = models.DateTimeField(
        auto_now=True  # Set on every save
    )  # DATETIME or TIMESTAMP
    
    published_at = models.DateTimeField(
        default=timezone.now,
        null=True,
        blank=True
    )  # DATETIME
```

### **8. DecimalField** *(Fixed-precision decimal)*

```python
class Product(models.Model):
    price = models.DecimalField(
        max_digits=10,      # Total digits (including decimals)
        decimal_places=2,   # Digits after decimal point
        default=0.00,
        validators=[MinValueValidator(0)]
    )  # DECIMAL(10, 2) or NUMERIC(10, 2)
    
    tax_rate = models.DecimalField(
        max_digits=5,
        decimal_places=2,
        default=0.00
    )  # DECIMAL(5, 2)
    
    # max_digits must be >= decimal_places
    weight = models.DecimalField(max_digits=8, decimal_places=3)  # e.g., 12345.678
```

### **9. DurationField** *(Time duration)*

```python
class Course(models.Model):
    duration = models.DurationField(
        default=timedelta(hours=1)
    )  # BIGINT (microseconds) or INTERVAL
    
    # Store durations: timedelta(days=1, hours=2, minutes=30)
    # Python: timedelta objects
```

### **10. EmailField** *(CharField with email validation)*

```python
class Contact(models.Model):
    email = models.EmailField(
        max_length=254,  # Standard email max length per RFC 5321
        unique=True
    )  # VARCHAR(254)
```

### **11. FileField** *(File upload)*

```python
class Document(models.Model):
    file = models.FileField(
        upload_to='documents/%Y/%m/%d/',  # Upload directory with date-based path
        max_length=100,                    # Max filename length
        storage=FileSystemStorage(),       # Storage backend
        blank=True,
        null=True
    )  # VARCHAR(100)
```

### **12. FilePathField** *(File selection from directory)*

```python
class Template(models.Model):
    template_file = models.FilePathField(
        path='/path/to/templates/',  # Required: absolute path
        match=r'.*\.html$',          # Regex pattern to filter filenames
        recursive=True,               # Include subdirectories
        allow_files=True,             # Allow files
        allow_folders=False,          # Allow folders
        max_length=100
    )  # VARCHAR(100)
    
    # Useful for selecting from existing files on the filesystem
```

### **13. FloatField** *(Floating-point number)*

```python
class Sensor(models.Model):
    temperature = models.FloatField(
        default=0.0
    )  # DOUBLE PRECISION or FLOAT
    
    humidity = models.FloatField()  # DOUBLE
    
    # Warning: Use DecimalField for monetary values to avoid rounding errors
```

### **14. GenericIPAddressField** *(IPv4 or IPv6 address)*

```python
class Server(models.Model):
    ip_address = models.GenericIPAddressField(
        protocol='both',     # 'both', 'IPv4', 'IPv6'
        unpack_ipv4=False,   # Map to IPv4-compatible IPv6 address
        blank=True,
        null=True
    )  # VARCHAR(39)
    
    ipv4 = models.GenericIPAddressField(protocol='IPv4')  # VARCHAR(15)
    ipv6 = models.GenericIPAddressField(protocol='IPv6')  # VARCHAR(39)
```

### **15. ImageField** *(FileField with image validation)*

```python
class Profile(models.Model):
    avatar = models.ImageField(
        upload_to='avatars/',
        height_field=None,          # Model field to store height
        width_field=None,           # Model field to store width
        max_length=100,
        validators=[validate_image_file_extension]  # Custom validation
    )  # VARCHAR(100)
    
    avatar_height = models.IntegerField(editable=False, null=True)
    avatar_width = models.IntegerField(editable=False, null=True)
    
    # Requires Pillow library: pip install Pillow
```

### **16. IntegerField** *(Standard integer)*

```python
class Product(models.Model):
    quantity = models.IntegerField(
        default=0,
        validators=[MinValueValidator(0), MaxValueValidator(999)]
    )  # INT
    # Range: -2147483648 to 2147483647
```

### **17. JSONField** *(JSON data)*

```python
class Configuration(models.Model):
    settings = models.JSONField(
        default=dict,      # Default empty dict
        encoder=DjangoJSONEncoder,  # Custom JSON encoder
        decoder=None,      # Custom JSON decoder
        blank=True,
        null=True
    )  # JSON or JSONB
    
    metadata = models.JSONField(
        default=list,      # Default empty list
    )  # JSON
    
    # Supports PostgreSQL JSON querying
    # Django 3.1+ supports all databases
```

### **18. PositiveBigIntegerField** *(0 to 9223372036854775807)*

```python
class ViewCounter(models.Model):
    total_views = models.PositiveBigIntegerField(
        default=0
    )  # BIGINT UNSIGNED
    # Range: 0 to 9223372036854775807
```

### **19. PositiveIntegerField** *(0 to 2147483647)*

```python
class Order(models.Model):
    quantity = models.PositiveIntegerField(
        default=1,
        validators=[MinValueValidator(1)]
    )  # INT UNSIGNED
    # Range: 0 to 2147483647
```

### **20. PositiveSmallIntegerField** *(0 to 32767)*

```python
class Rating(models.Model):
    score = models.PositiveSmallIntegerField(
        default=5,
        validators=[MinValueValidator(1), MaxValueValidator(5)]
    )  # SMALLINT UNSIGNED
    # Range: 0 to 32767
```

### **21. SlugField** *(URL-friendly string)*

```python
class Post(models.Model):
    slug = models.SlugField(
        max_length=100,
        allow_unicode=True,  # Support unicode characters (e.g., Chinese)
        unique=True,
        db_index=True
    )  # VARCHAR(100)
    
    # Automatically generated from title, e.g., "My First Post" → "my-first-post"
```

### **22. SmallAutoField** *(Auto-incrementing 16-bit integer, Django 3.0+)*

```python
class Category(models.Model):
    # For small tables (max 32767 records)
    category_id = models.SmallAutoField(primary_key=True)  # SMALLINT AUTO_INCREMENT
```

### **23. SmallIntegerField** *(16-bit integer)*

```python
class Priority(models.Model):
    level = models.SmallIntegerField(
        default=0,
        choices=[
            (1, 'Low'),
            (2, 'Medium'),
            (3, 'High'),
        ]
    )  # SMALLINT
    # Range: -32768 to 32767
```

### **24. TextField** *(Unlimited text)*

```python
class Article(models.Model):
    content = models.TextField(
        blank=True,
        db_collation='utf8mb4_general_ci',  # Database collation
        db_index=False  # Usually not indexed for large text
    )  # TEXT or LONGTEXT
    
    summary = models.TextField(
        max_length=500,  # Limit length in forms (not DB)
        blank=True
    )
```

### **25. TimeField** *(Time only, no date)*

```python
class Schedule(models.Model):
    start_time = models.TimeField(
        auto_now=False,
        auto_now_add=False,
        default='09:00:00'
    )  # TIME
    
    end_time = models.TimeField(
        null=True,
        blank=True
    )  # TIME
```

### **26. URLField** *(CharField with URL validation)*

```python
class SocialLink(models.Model):
    url = models.URLField(
        max_length=200,
        default='https://example.com',
        blank=True,
        assumptions_scheme='https'  # Add scheme if missing
    )  # VARCHAR(200)
```

### **27. UUIDField** *(Universally Unique Identifier)*

```python
import uuid
from django.db import models

class Transaction(models.Model):
    id = models.UUIDField(
        primary_key=True,
        default=uuid.uuid4,  # Generate random UUID
        editable=False       # Cannot be changed
    )  # UUID or CHAR(32)
    
    reference_id = models.UUIDField(
        default=uuid.uuid4,
        unique=True
    )  # UUID
    
    # UUID format: 550e8400-e29b-41d4-a716-446655440000
```

---

## **Relationship Fields**

### **28. ForeignKey** *(One-to-Many relationship)*

```python
class Author(models.Model):
    name = models.CharField(max_length=100)

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(
        Author,
        on_delete=models.CASCADE,        # Delete books when author is deleted
        related_name='books',            # author.books.all()
        related_query_name='book',       # Author.objects.filter(book__title='...')
        limit_choices_to={'is_active': True},  # Limit choices in forms
        db_constraint=True,              # Create foreign key constraint in DB
        to_field='id',                   # Field in referenced model
        swappable=True,                  # Allow swapping the model
        null=True,
        blank=True
    )  # INT REFERENCES author(id)
    
    # on_delete options:
    # CASCADE: Delete related objects
    # PROTECT: Prevent deletion if related objects exist
    # RESTRICT: Similar to PROTECT but with deferred checking (Django 3.1+)
    # SET_NULL: Set to NULL (requires null=True)
    # SET_DEFAULT: Set to default value (requires default)
    # SET(): Set to a specific value or callable
    # DO_NOTHING: Do nothing (may cause integrity errors)
```

### **29. ManyToManyField** *(Many-to-Many relationship)*

```python
class Student(models.Model):
    name = models.CharField(max_length=100)

class Course(models.Model):
    title = models.CharField(max_length=200)
    students = models.ManyToManyField(
        Student,
        related_name='courses',          # student.courses.all()
        related_query_name='course',     # Student.objects.filter(course__title='...')
        limit_choices_to={'is_active': True},
        symmetrical=True,                # For self-referential (False for non-symmetrical)
        through='Enrollment',            # Custom intermediate table
        through_fields=('course', 'student'),  # Fields for through table
        db_table='course_students',      # Custom junction table name
        blank=True
    )  # Creates intermediate table
    
    # Self-referential example
    friends = models.ManyToManyField(
        'self',
        symmetrical=True,  # If A is friend with B, B is friend with A
        blank=True
    )
```

### **30. OneToOneField** *(One-to-One relationship)*

```python
class User(models.Model):
    name = models.CharField(max_length=100)

class UserProfile(models.Model):
    user = models.OneToOneField(
        User,
        on_delete=models.CASCADE,
        primary_key=False,     # If True, uses as primary key (multi-table inheritance)
        parent_link=False,     # If True, used as parent link for inheritance
        related_name='profile'
    )  # INT UNIQUE REFERENCES user(id)
    
    bio = models.TextField()
    avatar = models.ImageField(upload_to='avatars/')
```

---

## **Custom Field Parameters (Common to All Fields)**

```python
class Example(models.Model):
    field_name = models.CharField(
        max_length=100,
        
        # Core parameters
        null=True,           # Allow NULL in database (default: False)
        blank=True,          # Allow empty in forms (default: False)
        choices=[],          # List of tuples for choices
        db_column='col',     # Custom column name in database
        db_index=True,       # Create database index
        db_tablespace='',    # Tablespace for field index
        default='',          # Default value or callable
        editable=True,       # Field can be edited in forms/admin
        error_messages={},   # Custom error messages
        help_text='',        # Help text for forms
        primary_key=False,   # Use as primary key
        unique=False,        # Must be unique
        unique_for_date='',  # Unique for date field
        unique_for_month='', # Unique for month field
        unique_for_year='',  # Unique for year field
        verbose_name='',     # Human-readable name
        validators=[],       # List of validators
    )
```

---

## **Field Type Compatibility Matrix**

```python
# Database-specific considerations:
# SQLite: Most fields work, but some types are limited
# PostgreSQL: All fields work natively, best JSONField support
# MySQL/MariaDB: Most fields work, check version for JSONField
# Oracle: All fields supported with some limitations

# Example of database-specific field usage:
class CrossDB(models.Model):
    # Works everywhere
    name = models.CharField(max_length=100)  # All databases
    count = models.IntegerField()            # All databases
    
    # PostgreSQL-specific (may need conditional usage)
    # Use JSONField carefully with MySQL < 5.7
    
    # Universal with caveats
    duration = models.DurationField()  # SQLite stores as BIGINT
    decimal = models.DecimalField(max_digits=10, decimal_places=2)  # All
```

---

## **Complete Example with All Field Types**

```python
from django.db import models
from django.core.validators import MinValueValidator, MaxValueValidator
from django.utils import timezone
import uuid
from datetime import timedelta

class AllFieldsExample(models.Model):
    # Auto-incrementing fields
    auto_id = models.AutoField(primary_key=True)
    big_auto_id = models.BigAutoField()
    small_auto_id = models.SmallAutoField()
    
    # Integer fields
    integer = models.IntegerField(default=0)                    # INT
    big_integer = models.BigIntegerField(default=0)             # BIGINT
    small_integer = models.SmallIntegerField(default=0)         # SMALLINT
    positive_integer = models.PositiveIntegerField(default=1)   # INT UNSIGNED
    positive_big_integer = models.PositiveBigIntegerField(default=1) # BIGINT UNSIGNED
    positive_small_integer = models.PositiveSmallIntegerField(default=1) # SMALLINT UNSIGNED
    
    # Boolean
    boolean = models.BooleanField(default=True)                 # BOOL
    
    # String fields
    char = models.CharField(max_length=255)                    # VARCHAR(255)
    text = models.TextField()                                   # TEXT
    slug = models.SlugField(max_length=100)                    # VARCHAR(100)
    email = models.EmailField(max_length=254)                  # VARCHAR(254)
    url = models.URLField(max_length=200)                       # VARCHAR(200)
    
    # Numeric fields
    decimal = models.DecimalField(max_digits=10, decimal_places=2) # DECIMAL(10,2)
    float = models.FloatField(default=0.0)                      # DOUBLE
    
    # Date/Time fields
    date = models.DateField()                                   # DATE
    time = models.TimeField()                                   # TIME
    datetime = models.DateTimeField()                           # DATETIME
    duration = models.DurationField()                           # BIGINT/INTERVAL
    
    # File fields
    file = models.FileField(upload_to='files/')                # VARCHAR
    file_path = models.FilePathField(path='/tmp/')             # VARCHAR
    image = models.ImageField(upload_to='images/')             # VARCHAR
    binary = models.BinaryField()                               # BLOB
    
    # Special fields
    ip_address = models.GenericIPAddressField()                # VARCHAR(39)
    uuid = models.UUIDField(default=uuid.uuid4)                # UUID
    json = models.JSONField(default=dict)                       # JSON
    
    class Meta:
        verbose_name = "All Fields Example"
        verbose_name_plural = "All Fields Examples"
```

