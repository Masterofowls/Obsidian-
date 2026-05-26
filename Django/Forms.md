

## Form Basics

### Creating Simple Forms

```python
# forms.py
from django import forms
from django.core.validators import MinLengthValidator, MaxLengthValidator

class ContactForm(forms.Form):
    name = forms.CharField(
        max_length=100,
        label='Your Name',
        initial='Enter your name',
        help_text='Please enter your full name'
    )
    email = forms.EmailField(
        label='Email Address',
        required=True
    )
    subject = forms.CharField(max_length=200)
    message = forms.CharField(
        widget=forms.Textarea,
        help_text='Enter your message here'
    )
    cc_myself = forms.BooleanField(
        required=False,
        initial=True,
        label='Send me a copy'
    )
    priority = forms.ChoiceField(
        choices=[
            ('low', 'Low'),
            ('medium', 'Medium'),
            ('high', 'High'),
        ],
        initial='medium'
    )

# Using form in view
def contact_view(request):
    if request.method == 'POST':
        form = ContactForm(request.POST)
        if form.is_valid():
            # Process cleaned data
            name = form.cleaned_data['name']
            email = form.cleaned_data['email']
            subject = form.cleaned_data['subject']
            message = form.cleaned_data['message']
            # Send email, save to DB, etc.
            return redirect('success')
    else:
        form = ContactForm()
    
    return render(request, 'contact.html', {'form': form})
```

### Form Class Methods

```python
class AdvancedForm(forms.Form):
    name = forms.CharField(max_length=100)
    
    # Override __init__ to customize form
    def __init__(self, *args, **kwargs):
        # Get custom parameters
        self.user = kwargs.pop('user', None)
        super().__init__(*args, **kwargs)
        
        # Customize fields based on user
        if self.user and self.user.is_staff:
            self.fields['name'].widget.attrs.update({'class': 'staff-input'})
        
        # Dynamic field ordering
        self.order_fields(['email', 'name', 'message'])
    
    # Clean entire form
    def clean(self):
        cleaned_data = super().clean()
        name = cleaned_data.get('name')
        email = cleaned_data.get('email')
        
        if name and email:
            if name.lower() in email.lower():
                raise forms.ValidationError(
                    "Name should not be part of email address"
                )
        
        return cleaned_data
```

---

## Complete Form Fields Reference

### String Fields

```python
class StringFieldsForm(forms.Form):
    # CharField
    first_name = forms.CharField(
        max_length=100,
        min_length=2,
        strip=True,          # Strip whitespace
        empty_value='',      # Value to use when empty
        required=True,
        label='First Name',
        initial='John',
        help_text='Enter your first name'
    )
    
    # RegexField
    phone = forms.RegexField(
        regex=r'^\+?1?\d{9,15}$',
        error_messages={
            'invalid': 'Enter a valid phone number'
        },
        help_text='Format: +999999999'
    )
    
    # SlugField
    slug = forms.SlugField(
        max_length=100,
        allow_unicode=False
    )
    
    # URLField
    website = forms.URLField(
        max_length=200,
        assume_scheme='https',  # Add scheme if missing
        required=False
    )
    
    # UUIDField
    uuid = forms.UUIDField()
```

### Numeric Fields

```python
class NumericFieldsForm(forms.Form):
    # IntegerField
    age = forms.IntegerField(
        min_value=0,
        max_value=150,
        required=True,
        error_messages={
            'min_value': 'Age must be positive',
            'max_value': 'Age cannot exceed 150'
        }
    )
    
    # DecimalField
    price = forms.DecimalField(
        max_digits=10,
        decimal_places=2,
        min_value=0.01,
        max_value=99999999.99,
        localize=True  # Use locale-specific formatting
    )
    
    # FloatField
    rating = forms.FloatField(
        min_value=0.0,
        max_value=5.0,
        required=False,
        widget=forms.NumberInput(attrs={'step': 0.1})
    )
```

### Choice Fields

```python
class ChoiceFieldsForm(forms.Form):
    # ChoiceField
    GENDER_CHOICES = [
        ('M', 'Male'),
        ('F', 'Female'),
        ('O', 'Other'),
    ]
    gender = forms.ChoiceField(
        choices=GENDER_CHOICES,
        widget=forms.RadioSelect,
        initial='O'
    )
    
    # MultipleChoiceField
    INTEREST_CHOICES = [
        ('tech', 'Technology'),
        ('sports', 'Sports'),
        ('music', 'Music'),
        ('art', 'Art'),
        ('food', 'Food'),
    ]
    interests = forms.MultipleChoiceField(
        choices=INTEREST_CHOICES,
        widget=forms.CheckboxSelectMultiple,
        required=False
    )
    
    # TypedChoiceField
    ANSWER_CHOICES = [
        (1, 'Option A'),
        (2, 'Option B'),
        (3, 'Option C'),
    ]
    answer = forms.TypedChoiceField(
        choices=ANSWER_CHOICES,
        coerce=int,  # Convert value to int
        empty_value=None
    )
    
    # TypedMultipleChoiceField
    selected_options = forms.TypedMultipleChoiceField(
        choices=[(1, 'A'), (2, 'B'), (3, 'C')],
        coerce=int,
        required=False
    )
```

### Date/Time Fields

```python
class DateTimeFieldsForm(forms.Form):
    # DateField
    birth_date = forms.DateField(
        widget=forms.DateInput(attrs={'type': 'date'}),
        input_formats=['%Y-%m-%d', '%m/%d/%Y'],  # Accepted formats
        required=True
    )
    
    # DateTimeField
    appointment = forms.DateTimeField(
        widget=forms.DateTimeInput(attrs={'type': 'datetime-local'}),
        input_formats=['%Y-%m-%dT%H:%M', '%Y-%m-%d %H:%M:%S']
    )
    
    # TimeField
    meeting_time = forms.TimeField(
        widget=forms.TimeInput(attrs={'type': 'time'}),
        input_formats=['%H:%M', '%I:%M %p']
    )
    
    # DurationField
    duration = forms.DurationField(
        help_text='Format: DD HH:MM:SS',
        required=False
    )
    
    # SplitDateTimeField
    event_datetime = forms.SplitDateTimeField(
        widget=forms.SplitDateTimeWidget(
            date_attrs={'type': 'date'},
            time_attrs={'type': 'time'}
        ),
        input_date_formats=['%Y-%m-%d'],
        input_time_formats=['%H:%M']
    )
```

### File Fields

```python
class FileFieldsForm(forms.Form):
    # FileField
    document = forms.FileField(
        label='Upload Document',
        allow_empty_file=False,
        max_length=100,
        required=True,
        help_text='Max size: 5MB'
    )
    
    # ImageField
    avatar = forms.ImageField(
        label='Profile Picture',
        required=False,
        help_text='Upload a square image (jpg/png)',
        error_messages={
            'invalid_image': 'Please upload a valid image file'
        }
    )
    
    # FilePathField
    template = forms.FilePathField(
        path='/var/www/templates/',
        match='.*\.html$',
        recursive=False,
        required=False,
        help_text='Select a template file'
    )
```

### Boolean Fields

```python
class BooleanFieldsForm(forms.Form):
    # BooleanField
    agree_terms = forms.BooleanField(
        required=True,
        label='I agree to the terms and conditions',
        initial=False,
        error_messages={
            'required': 'You must agree to the terms to continue'
        }
    )
    
    # NullBooleanField
    newsletter = forms.NullBooleanField(
        label='Subscribe to newsletter?',
        widget=forms.NullBooleanSelect,
        initial=None
    )
```

### Advanced Fields

```python
class AdvancedFieldsForm(forms.Form):
    # ComboField (multiple validators)
    username = forms.ComboField(
        fields=[
            forms.CharField(max_length=30),
            forms.RegexField(r'^[a-zA-Z0-9_]+$')
        ],
        help_text='Letters, numbers, and underscores only'
    )
    
    # MultiValueField (multiple input fields)
    class PhoneWidget(forms.MultiWidget):
        def __init__(self):
            widgets = [
                forms.TextInput(attrs={'placeholder': 'Area Code', 'size': '3'}),
                forms.TextInput(attrs={'placeholder': 'Number', 'size': '7'})
            ]
            super().__init__(widgets)
        
        def decompress(self, value):
            if value:
                return value.split('-')
            return [None, None]
    
    class PhoneField(forms.MultiValueField):
        def __init__(self):
            fields = [
                forms.CharField(max_length=3),
                forms.CharField(max_length=7)
            ]
            super().__init__(fields, widget=PhoneWidget())
        
        def compress(self, data_list):
            if data_list:
                return '-'.join(data_list)
            return ''
    
    phone = PhoneField()
    
    # SplitDateTimeField (already covered)
    # JSONField (Django 3.1+)
    settings = forms.JSONField(
        required=False,
        encoder=None,
        decoder=None,
        widget=forms.Textarea(attrs={'rows': 5}),
        help_text='Enter valid JSON'
    )
```

---

## Complete Widgets Reference

### Basic Input Widgets

```python
class WidgetForm(forms.Form):
    # TextInput
    username = forms.CharField(
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'Enter username',
            'maxlength': 50,
            'autocomplete': 'off',
            'readonly': False,
            'disabled': False,
            'size': 30,
        })
    )
    
    # NumberInput
    quantity = forms.IntegerField(
        widget=forms.NumberInput(attrs={
            'min': 1,
            'max': 100,
            'step': 1,
            'class': 'quantity-input'
        })
    )
    
    # EmailInput
    email = forms.EmailField(
        widget=forms.EmailInput(attrs={
            'placeholder': 'user@example.com',
            'multiple': True  # Allow multiple emails
        })
    )
    
    # URLInput
    website = forms.URLField(
        widget=forms.URLInput(attrs={
            'placeholder': 'https://example.com'
        })
    )
    
    # PasswordInput
    password = forms.CharField(
        widget=forms.PasswordInput(attrs={
            'placeholder': 'Enter password',
            'render_value': False  # Don't show password on validation error
        })
    )
    
    # HiddenInput
    user_id = forms.IntegerField(
        widget=forms.HiddenInput()
    )
    
    # Textarea
    message = forms.CharField(
        widget=forms.Textarea(attrs={
            'rows': 5,
            'cols': 40,
            'placeholder': 'Type your message...',
            'class': 'form-control',
            'style': 'resize: vertical;'
        })
    )
```

### Choice Widgets

```python
class ChoiceWidgetsForm(forms.Form):
    GENDER_CHOICES = [('M', 'Male'), ('F', 'Female'), ('O', 'Other')]
    COLOR_CHOICES = [('red', 'Red'), ('blue', 'Blue'), ('green', 'Green')]
    
    # Select (default for ChoiceField)
    country = forms.ChoiceField(
        choices=[('us', 'USA'), ('uk', 'UK'), ('ca', 'Canada')],
        widget=forms.Select(attrs={
            'class': 'form-select',
            'onchange': 'handleCountryChange(this)'
        })
    )
    
    # SelectMultiple
    languages = forms.MultipleChoiceField(
        choices=[('en', 'English'), ('es', 'Spanish'), ('fr', 'French')],
        widget=forms.SelectMultiple(attrs={
            'size': 5,
            'class': 'form-select'
        })
    )
    
    # RadioSelect
    gender = forms.ChoiceField(
        choices=GENDER_CHOICES,
        widget=forms.RadioSelect(attrs={
            'class': 'form-check-input'
        })
    )
    
    # CheckboxSelectMultiple
    favorite_colors = forms.MultipleChoiceField(
        choices=COLOR_CHOICES,
        widget=forms.CheckboxSelectMultiple(attrs={
            'class': 'form-check-input'
        })
    )
    
    # NullBooleanSelect
    newsletter = forms.NullBooleanField(
        widget=forms.NullBooleanSelect(attrs={
            'class': 'form-select'
        })
    )
    
    # SelectDateWidget
    birth_date = forms.DateField(
        widget=forms.SelectDateWidget(
            years=range(1950, 2025),
            empty_label=('Year', 'Month', 'Day'),
            attrs={'class': 'form-select'}
        )
    )
```

### File Widgets

```python
class FileWidgetsForm(forms.Form):
    # FileInput
    document = forms.FileField(
        widget=forms.FileInput(attrs={
            'class': 'form-control',
            'accept': '.pdf,.doc,.docx',
            'multiple': False  # Single file
        })
    )
    
    # ClearableFileInput (default for FileField)
    avatar = forms.ImageField(
        widget=forms.ClearableFileInput(attrs={
            'class': 'form-control',
            'accept': 'image/*',
            'multiple': False
        }),
        required=False
    )
```

### DateTime Widgets

```python
class DateTimeWidgetsForm(forms.Form):
    # DateInput
    start_date = forms.DateField(
        widget=forms.DateInput(
            attrs={'type': 'date', 'class': 'form-control'},
            format='%Y-%m-%d'
        )
    )
    
    # DateTimeInput
    appointment = forms.DateTimeField(
        widget=forms.DateTimeInput(
            attrs={'type': 'datetime-local', 'class': 'form-control'},
            format='%Y-%m-%dT%H:%M'
        )
    )
    
    # TimeInput
    meeting_time = forms.TimeField(
        widget=forms.TimeInput(
            attrs={'type': 'time', 'class': 'form-control'},
            format='%H:%M'
        )
    )
    
    # SplitDateTimeWidget
    event_datetime = forms.SplitDateTimeField(
        widget=forms.SplitDateTimeWidget(
            date_attrs={'type': 'date', 'class': 'form-control'},
            time_attrs={'type': 'time', 'class': 'form-control'}
        )
    )
```

### Compound Widgets

```python
class CompoundWidgetForm(forms.Form):
    # SplitHiddenDateTimeWidget
    hidden_datetime = forms.SplitDateTimeField(
        widget=forms.SplitHiddenDateTimeWidget()
    )
    
    # MultiWidget (custom compound widget)
    class CreditCardWidget(forms.MultiWidget):
        def __init__(self):
            widgets = [
                forms.TextInput(attrs={'placeholder': 'XXXX', 'size': 4}),
                forms.TextInput(attrs={'placeholder': 'XXXX', 'size': 4}),
                forms.TextInput(attrs={'placeholder': 'XXXX', 'size': 4}),
                forms.TextInput(attrs={'placeholder': 'XXXX', 'size': 4}),
            ]
            super().__init__(widgets)
        
        def decompress(self, value):
            if value:
                return value.split('-')
            return [None, None, None, None]
    
    credit_card = forms.CharField(
        widget=CreditCardWidget(),
        max_length=19
    )
```

---

## Form Validation

### Field-Level Validation

```python
class RegistrationForm(forms.Form):
    username = forms.CharField(max_length=100)
    email = forms.EmailField()
    password = forms.CharField(widget=forms.PasswordInput)
    confirm_password = forms.CharField(widget=forms.PasswordInput)
    age = forms.IntegerField()
    
    # Custom field validation - clean_<fieldname>
    def clean_username(self):
        username = self.cleaned_data['username']
        
        # Check for existing username
        if User.objects.filter(username=username).exists():
            raise forms.ValidationError("Username already exists")
        
        # Check length
        if len(username) < 3:
            raise forms.ValidationError("Username must be at least 3 characters")
        
        # Check for valid characters
        if not username.isalnum():
            raise forms.ValidationError("Username must be alphanumeric")
        
        return username.lower()  # Normalize to lowercase
    
    def clean_email(self):
        email = self.cleaned_data['email']
        
        # Check for existing email
        if User.objects.filter(email=email).exists():
            raise forms.ValidationError("Email already registered")
        
        # Check for disposable email domains
        disposable_domains = ['mailinator.com', 'guerrillamail.com']
        domain = email.split('@')[1]
        if domain in disposable_domains:
            raise forms.ValidationError("Disposable emails are not allowed")
        
        return email
    
    def clean_password(self):
        password = self.cleaned_data['password']
        
        # Password strength validation
        if len(password) < 8:
            raise forms.ValidationError("Password must be at least 8 characters")
        
        if not any(c.isupper() for c in password):
            raise forms.ValidationError("Password must contain an uppercase letter")
        
        if not any(c.islower() for c in password):
            raise forms.ValidationError("Password must contain a lowercase letter")
        
        if not any(c.isdigit() for c in password):
            raise forms.ValidationError("Password must contain a number")
        
        return password
    
    def clean_age(self):
        age = self.cleaned_data['age']
        
        if age < 18:
            raise forms.ValidationError("You must be at least 18 years old")
        
        if age > 120:
            raise forms.ValidationError("Please enter a valid age")
        
        return age
```

### Cross-Field Validation

```python
class OrderForm(forms.Form):
    quantity = forms.IntegerField(min_value=1)
    unit_price = forms.DecimalField(max_digits=10, decimal_places=2)
    discount_code = forms.CharField(required=False)
    shipping_method = forms.ChoiceField(
        choices=[('standard', 'Standard'), ('express', 'Express')]
    )
    delivery_date = forms.DateField(required=False)
    
    # Cross-field validation in clean() method
    def clean(self):
        cleaned_data = super().clean()
        quantity = cleaned_data.get('quantity')
        unit_price = cleaned_data.get('unit_price')
        discount_code = cleaned_data.get('discount_code')
        shipping_method = cleaned_data.get('shipping_method')
        delivery_date = cleaned_data.get('delivery_date')
        
        # Validate total price
        if quantity and unit_price:
            total = quantity * unit_price
            if total > 10000:
                raise forms.ValidationError(
                    "Order total cannot exceed $10,000"
                )
        
        # Validate discount code
        if discount_code:
            if not DiscountCode.objects.filter(
                code=discount_code, is_active=True
            ).exists():
                self.add_error('discount_code', 'Invalid discount code')
        
        # Validate shipping and delivery
        if shipping_method == 'express' and not delivery_date:
            self.add_error(
                'delivery_date',
                'Delivery date is required for express shipping'
            )
        
        return cleaned_data
    
    # Dependent field validation
    def clean_delivery_date(self):
        delivery_date = self.cleaned_data.get('delivery_date')
        shipping_method = self.cleaned_data.get('shipping_method')
        
        if shipping_method == 'express' and delivery_date:
            from datetime import date, timedelta
            min_date = date.today() + timedelta(days=1)
            if delivery_date < min_date:
                raise forms.ValidationError(
                    "Express delivery must be at least 1 day from now"
                )
        
        return delivery_date
```

### Custom Validators

```python
from django.core.exceptions import ValidationError
from django.core.validators import BaseValidator
from django.utils.deconstruct import deconstructible
import re

# Simple function validator
def validate_even(value):
    if value % 2 != 0:
        raise ValidationError(
            '%(value)s is not an even number',
            params={'value': value}
        )

def validate_file_size(value):
    filesize = value.size
    max_size = 5 * 1024 * 1024  # 5MB
    if filesize > max_size:
        raise ValidationError('File size cannot exceed 5MB')

# Class-based validator
@deconstructible
class PhoneNumberValidator:
    def __init__(self, country_code='+1'):
        self.country_code = country_code
    
    def __call__(self, value):
        pattern = r'^\+?\d{10,15}$'
        if not re.match(pattern, value):
            raise ValidationError('Enter a valid phone number')
    
    def __eq__(self, other):
        return self.country_code == other.country_code

# Validator with parameters
@deconstructible
class AgeValidator:
    def __init__(self, min_age=18, max_age=120):
        self.min_age = min_age
        self.max_age = max_age
    
    def __call__(self, value):
        if value < self.min_age:
            raise ValidationError(
                f'Age must be at least {self.min_age} years'
            )
        if value > self.max_age:
            raise ValidationError(
                f'Age cannot exceed {self.max_age} years'
            )

# Using custom validators
class CustomValidatorForm(forms.Form):
    even_number = forms.IntegerField(validators=[validate_even])
    phone = forms.CharField(
        max_length=15,
        validators=[PhoneNumberValidator(country_code='+1')]
    )
    age = forms.IntegerField(
        validators=[AgeValidator(min_age=18, max_age=120)]
    )
    document = forms.FileField(
        validators=[validate_file_size]
    )
```

---

## ModelForms

### Basic ModelForm

```python
from django.forms import ModelForm
from .models import Post, Comment, UserProfile

class PostForm(ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'content', 'category', 'tags', 'status']
        # Or use:
        # fields = '__all__'
        # exclude = ['author', 'created_at', 'views']
        
        # Custom widgets
        widgets = {
            'title': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': 'Enter title'
            }),
            'content': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 10
            }),
            'category': forms.Select(attrs={'class': 'form-select'}),
            'tags': forms.CheckboxSelectMultiple(),
            'status': forms.RadioSelect()
        }
        
        # Custom labels
        labels = {
            'title': 'Post Title',
            'content': 'Post Content',
        }
        
        # Help texts
        help_texts = {
            'title': 'Choose a descriptive title',
            'tags': 'Select relevant tags for your post',
        }
        
        # Error messages
        error_messages = {
            'title': {
                'max_length': 'Title is too long',
                'required': 'Please enter a title',
            }
        }
    
    # Add custom fields
    tags_input = forms.CharField(
        required=False,
        widget=forms.TextInput(attrs={
            'placeholder': 'Add tags separated by commas'
        })
    )
    
    # Override init to customize
    def __init__(self, *args, **kwargs):
        self.user = kwargs.pop('user', None)
        super().__init__(*args, **kwargs)
        
        # Make certain fields optional/required based on context
        if self.user and not self.user.is_staff:
            self.fields['status'].queryset = Status.objects.filter(
                name__in=['draft', 'review']
            )
        
        # Add CSS classes to all fields
        for field_name, field in self.fields.items():
            if field_name != 'tags':
                field.widget.attrs.update({'class': 'form-control'})
    
    # Custom field validation
    def clean_title(self):
        title = self.cleaned_data['title']
        
        # Check for duplicate titles (excluding current instance)
        queryset = Post.objects.filter(title__iexact=title)
        if self.instance.pk:
            queryset = queryset.exclude(pk=self.instance.pk)
        
        if queryset.exists():
            raise forms.ValidationError("A post with this title already exists")
        
        return title
    
    # Custom save method
    def save(self, commit=True):
        instance = super().save(commit=False)
        
        # Set author if creating
        if not instance.pk and hasattr(self, 'user'):
            instance.author = self.user
        
        # Process custom fields
        if self.cleaned_data.get('tags_input'):
            tags = [tag.strip() for tag in self.cleaned_data['tags_input'].split(',')]
            # Process tags logic
        
        if commit:
            instance.save()
            self.save_m2m()  # Save many-to-many relationships
        
        return instance
```

### Advanced ModelForm Techniques

```python
class ProductForm(ModelForm):
    class Meta:
        model = Product
        fields = '__all__'
    
    # Override field definitions
    description = forms.CharField(
        widget=forms.Textarea(attrs={'rows': 5}),
        required=False
    )
    
    # Add fields not in model
    confirm_price = forms.DecimalField(
        max_digits=10,
        decimal_places=2,
        label='Confirm Price'
    )
    
    def clean(self):
        cleaned_data = super().clean()
        price = cleaned_data.get('price')
        confirm_price = cleaned_data.get('confirm_price')
        
        if price and confirm_price and price != confirm_price:
            self.add_error('confirm_price', 'Prices do not match')
        
        return cleaned_data

# ModelForm with foreign key fields
class CommentForm(ModelForm):
    class Meta:
        model = Comment
        fields = ['content', 'parent']
        widgets = {
            'content': forms.Textarea(attrs={
                'rows': 3,
                'placeholder': 'Write a comment...'
            }),
            'parent': forms.HiddenInput()  # For reply functionality
        }
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        # Only show active/published posts
        self.fields['post'].queryset = Post.objects.filter(
            status='published'
        )
```

---

## Formsets

### Basic Formsets

```python
from django.forms import formset_factory, modelformset_factory
from django.forms import inlineformset_factory, BaseFormSet

# Regular Formset
class ArticleForm(forms.Form):
    title = forms.CharField(max_length=100)

# Create formset
ArticleFormSet = formset_factory(
    ArticleForm,
    extra=2,           # Number of empty forms
    max_num=5,         # Maximum number of forms
    min_num=1,         # Minimum number of forms
    validate_min=True,  # Validate minimum
    validate_max=True,  # Validate maximum
    can_order=True,     # Allow ordering of forms
    can_delete=True,    # Allow deletion of forms
    absolute_max=10,    # Hard limit
)

# Using formset in view
def manage_articles(request):
    if request.method == 'POST':
        formset = ArticleFormSet(request.POST, prefix='articles')
        if formset.is_valid():
            for form in formset:
                if form.cleaned_data and not form.cleaned_data.get('DELETE', False):
                    title = form.cleaned_data.get('title')
                    if title:
                        Article.objects.create(title=title)
            return redirect('articles')
    else:
        formset = ArticleFormSet(prefix='articles')
    
    return render(request, 'manage_articles.html', {'formset': formset})
```

### ModelFormset

```python
from django.forms import modelformset_factory, BaseModelFormSet

class PostModelForm(ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'status']

# Create model formset
PostFormSet = modelformset_factory(
    Post,
    form=PostModelForm,
    fields=['title', 'status'],
    extra=1,
    can_delete=True,
    can_order=True,
)

# Custom model formset class
class RequiredPostFormSet(BaseModelFormSet):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.queryset = Post.objects.filter(status='published')
    
    def clean(self):
        super().clean()
        
        # Ensure at least one form is filled
        if any(self.errors):
            return
        
        titles = []
        for form in self.forms:
            if self.can_delete and self._should_delete_form(form):
                continue
            
            title = form.cleaned_data.get('title')
            if title:
                if title in titles:
                    raise forms.ValidationError(
                        "Posts must have unique titles"
                    )
                titles.append(title)

# Using model formset in view
def bulk_edit_posts(request):
    PostFormSet = modelformset_factory(
        Post,
        formset=RequiredPostFormSet,
        fields=['title', 'status'],
        extra=0,  # No empty forms
    )
    
    if request.method == 'POST':
        formset = PostFormSet(request.POST, queryset=Post.objects.filter(
            author=request.user
        ))
        if formset.is_valid():
            formset.save()
            return redirect('post_list')
    else:
        formset = PostFormSet(queryset=Post.objects.filter(
            author=request.user
        ))
    
    return render(request, 'bulk_edit.html', {'formset': formset})
```

### Inline Formsets

```python
from django.forms import inlineformset_factory

class OrderItemForm(ModelForm):
    class Meta:
        model = OrderItem
        fields = ['product', 'quantity', 'unit_price']
        widgets = {
            'product': forms.Select(attrs={'class': 'product-select'}),
            'quantity': forms.NumberInput(attrs={'min': 1, 'class': 'quantity-input'}),
            'unit_price': forms.NumberInput(attrs={'readonly': 'readonly'}),
        }

# Create inline formset
OrderItemFormSet = inlineformset_factory(
    Order,                    # Parent model
    OrderItem,               # Child model
    form=OrderItemForm,
    fields=['product', 'quantity', 'unit_price'],
    extra=1,                 # Extra empty forms
    can_delete=True,         # Allow deletion
    can_order=False,         # No ordering
    min_num=1,               # At least one item
    validate_min=True,
    fk_name='order',         # Foreign key field name
)

class OrderUpdateView(UpdateView):
    model = Order
    form_class = OrderForm
    template_name = 'orders/order_form.html'
    success_url = reverse_lazy('order_list')
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        if self.request.POST:
            context['items_formset'] = OrderItemFormSet(
                self.request.POST,
                instance=self.object,
                prefix='items'
            )
        else:
            context['items_formset'] = OrderItemFormSet(
                instance=self.object,
                prefix='items'
            )
        return context
    
    def form_valid(self, form):
        context = self.get_context_data()
        items_formset = context['items_formset']
        
        if items_formset.is_valid():
            self.object = form.save()
            items_formset.instance = self.object
            items_formset.save()
            return super().form_valid(form)
        
        return self.render_to_response(
            self.get_context_data(form=form)
        )
```

### Custom Formset Validation

```python
class BaseInvoiceItemFormSet(BaseFormSet):
    def clean(self):
        if any(self.errors):
            return
        
        quantities = []
        total_amount = 0
        
        for form in self.forms:
            if self.can_delete and self._should_delete_form(form):
                continue
            
            quantity = form.cleaned_data.get('quantity', 0)
            unit_price = form.cleaned_data.get('unit_price', 0)
            
            if quantity and unit_price:
                quantities.append(quantity)
                total_amount += quantity * unit_price
        
        # Validate total quantity
        if sum(quantities) == 0:
            raise forms.ValidationError("You must add at least one item")
        
        # Validate total amount
        if total_amount > 100000:
            raise forms.ValidationError(
                "Total invoice amount cannot exceed $100,000"
            )
        
        # Validate no duplicate products
        products = []
        for form in self.forms:
            if form.cleaned_data.get('product'):
                product = form.cleaned_data['product']
                if product in products:
                    raise forms.ValidationError(
                        f"Duplicate product: {product}"
                    )
                products.append(product)

# Using custom formset
InvoiceItemFormSet = inlineformset_factory(
    Invoice,
    InvoiceItem,
    formset=BaseInvoiceItemFormSet,
    fields=['product', 'quantity', 'unit_price'],
    extra=1
)
```

---

## Form Rendering

### Template Rendering Options

```html
<!-- Default form rendering -->
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}  <!-- as paragraphs -->
    {{ form.as_table }}  <!-- as table -->
    {{ form.as_ul }}  <!-- as unordered list -->
    {{ form.as_div }}  <!-- as divs (Django 4.1+) -->
    <button type="submit">Submit</button>
</form>

<!-- Manual field rendering -->
<form method="post">
    {% csrf_token %}
    
    <!-- Individual field -->
    <div class="field-wrapper">
        {{ form.subject.label_tag }}
        {{ form.subject }}
        {{ form.subject.errors }}
        {{ form.subject.help_text }}
    </div>
    
    <!-- Loop through fields -->
    {% for field in form %}
        <div class="form-group">
            <label for="{{ field.id_for_label }}">
                {{ field.label }}
                {% if field.field.required %}
                    <span class="required">*</span>
                {% endif %}
            </label>
            
            {{ field }}
            
            {% if field.help_text %}
                <small class="help-text">{{ field.help_text }}</small>
            {% endif %}
            
            {% if field.errors %}
                <div class="errors">
                    {{ field.errors }}
                </div>
            {% endif %}
        </div>
    {% endfor %}
    
    <button type="submit">Submit</button>
</form>

<!-- Custom field rendering with attributes -->
<div class="custom-field">
    <label for="{{ form.email.id_for_label }}">
        Email Address
    </label>
    <input type="email"
           name="{{ form.email.name }}"
           id="{{ form.email.id_for_label }}"
           value="{{ form.email.value|default:'' }}"
           class="form-control {% if form.email.errors %}is-invalid{% endif %}"
           placeholder="Enter your email">
    
    {% for error in form.email.errors %}
        <div class="invalid-feedback">{{ error }}</div>
    {% endfor %}
</div>

<!-- Rendering formsets -->
<form method="post">
    {% csrf_token %}
    {{ formset.management_form }}
    
    {% for form in formset %}
        <div class="formset-row">
            {{ form.id }}  <!-- Hidden ID field -->
            
            {% for field in form.visible_fields %}
                <div class="form-group">
                    {{ field.label_tag }}
                    {{ field }}
                    {{ field.errors }}
                </div>
            {% endfor %}
            
            {% if formset.can_delete %}
                <div class="delete-row">
                    {{ form.DELETE }} Delete
                </div>
            {% endif %}
        </div>
    {% endfor %}
    
    <button type="submit">Save All</button>
</form>
```

### Template Tags and Filters

```python
# Custom template tags (templatetags/form_tags.py)
from django import template

register = template.Library()

@register.filter
def add_class(field, css_class):
    """Add CSS class to form field"""
    return field.as_widget(attrs={'class': css_class})

@register.filter
def add_placeholder(field, placeholder):
    """Add placeholder to form field"""
    field.field.widget.attrs['placeholder'] = placeholder
    return field

@register.filter
def widget_type(field):
    """Get widget type"""
    return field.field.widget.__class__.__name__

@register.inclusion_tag('forms/form_field.html')
def render_field(field, **kwargs):
    """Custom field rendering"""
    return {
        'field': field,
        'custom_class': kwargs.get('class', ''),
        'show_help': kwargs.get('show_help', True),
        'show_label': kwargs.get('show_label', True),
    }
```

```html
<!-- Using custom tags in templates -->
{% load form_tags %}

<form method="post">
    {% csrf_token %}
    
    <!-- Add class to all fields -->
    <div class="form-group">
        {{ form.name|add_class:"form-control" }}
    </div>
    
    <!-- Add placeholder -->
    <div class="form-group">
        {{ form.email|add_placeholder:"Enter your email" }}
    </div>
    
    <!-- Render custom field -->
    {% render_field form.message class="custom-textarea" show_help=True %}
</form>
```

### Form Widget Customization

```python
class StyledForm(forms.Form):
    name = forms.CharField(
        widget=forms.TextInput(attrs={
            'class': 'form-control input-lg',
            'placeholder': 'Full Name',
            'data-validation': 'required',
            'aria-label': 'Full Name',
        })
    )
    
    email = forms.EmailField(
        widget=forms.EmailInput(attrs={
            'class': 'form-control',
            'placeholder': 'Email Address',
            'autocomplete': 'email',
            'pattern': '[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}$',
        })
    )
    
    # Custom widget class
    class DatePickerInput(forms.DateInput):
        template_name = 'widgets/datepicker.html'  # Custom template
        
        class Media:
            css = {
                'all': ('css/datepicker.css',)
            }
            js = ('js/datepicker.js',)
    
    birth_date = forms.DateField(
        widget=DatePickerInput(attrs={
            'class': 'datepicker',
            'data-date-format': 'yyyy-mm-dd',
        })
    )
```

---

## Advanced Form Techniques

### Dynamic Forms

```python
class DynamicForm(forms.Form):
    field_type = forms.ChoiceField(
        choices=[('text', 'Text'), ('number', 'Number'), ('date', 'Date')]
    )
    
    def __init__(self, *args, **kwargs):
        field_configs = kwargs.pop('field_configs', [])
        super().__init__(*args, **kwargs)
        
        # Dynamically add fields based on configuration
        for config in field_configs:
            field_name = config['name']
            field_type = config['type']
            
            if field_type == 'text':
                self.fields[field_name] = forms.CharField(
                    label=config.get('label', field_name),
                    required=config.get('required', True)
                )
            elif field_type == 'number':
                self.fields[field_name] = forms.IntegerField(
                    label=config.get('label', field_name),
                    min_value=config.get('min', 0),
                    max_value=config.get('max', 100)
                )
            elif field_type == 'choice':
                self.fields[field_name] = forms.ChoiceField(
                    choices=config.get('choices', []),
                    label=config.get('label', field_name)
                )

# Conditional fields
class ConditionalForm(forms.Form):
    has_shipping = forms.BooleanField(required=False, label='Different shipping address?')
    
    shipping_address = forms.CharField(
        required=False,
        widget=forms.Textarea(attrs={'rows': 3})
    )
    shipping_city = forms.CharField(required=False)
    shipping_zip = forms.CharField(required=False)
    
    def clean(self):
        cleaned_data = super().clean()
        has_shipping = cleaned_data.get('has_shipping')
        
        if has_shipping:
            shipping_fields = ['shipping_address', 'shipping_city', 'shipping_zip']
            for field in shipping_fields:
                if not cleaned_data.get(field):
                    self.add_error(field, 'This field is required')
        
        return cleaned_data
```

### Multi-Step Forms (Wizard)

```python
from django.forms import Form
from formtools.wizard.views import SessionWizardView

class StepOneForm(Form):
    name = forms.CharField(max_length=100)
    email = forms.EmailField()

class StepTwoForm(Form):
    address = forms.CharField(widget=forms.Textarea)
    phone = forms.CharField(max_length=15)

class StepThreeForm(Form):
    plan = forms.ChoiceField(choices=[
        ('basic', 'Basic'),
        ('premium', 'Premium'),
        ('enterprise', 'Enterprise'),
    ])
    terms = forms.BooleanField(required=True)

class SignupWizard(SessionWizardView):
    form_list = [StepOneForm, StepTwoForm, StepThreeForm]
    template_name = 'wizard/signup.html'
    
    def done(self, form_list, **kwargs):
        # Process all form data
        form_data = {}
        for form in form_list:
            form_data.update(form.cleaned_data)
        
        # Create user from all form data
        user = User.objects.create(
            username=form_data['name'],
            email=form_data['email'],
            # ... other fields
        )
        
        return redirect('signup_complete')
    
    def get_template_names(self):
        return [f'wizard/signup_step_{self.steps.step1}.html']
    
    def get_context_data(self, form, **kwargs):
        context = super().get_context_data(form=form, **kwargs)
        if self.steps.current == '0':
            context['progress'] = 33
        elif self.steps.current == '1':
            context['progress'] = 66
        else:
            context['progress'] = 100
        return context
```

### AJAX Form Handling

```python
# forms.py
class AJAXLoginForm(forms.Form):
    username = forms.CharField()
    password = forms.CharField(widget=forms.PasswordInput)
    remember = forms.BooleanField(required=False)
    
    def clean(self):
        cleaned_data = super().clean()
        username = cleaned_data.get('username')
        password = cleaned_data.get('password')
        
        if username and password:
            user = authenticate(username=username, password=password)
            if not user:
                raise forms.ValidationError("Invalid credentials")
            if not user.is_active:
                raise forms.ValidationError("Account is disabled")
        
        return cleaned_data

# views.py
class AJAXFormView(View):
    def post(self, request, *args, **kwargs):
        form = AJAXLoginForm(request.POST)
        
        if form.is_valid():
            # Process form
            return JsonResponse({
                'success': True,
                'redirect_url': reverse('dashboard')
            })
        
        # Return errors
        return JsonResponse({
            'success': False,
            'errors': form.errors,
            'field_errors': {
                field: [str(e) for e in errors]
                for field, errors in form.errors.items()
            }
        }, status=400)
```

```javascript
// AJAX form submission with JavaScript
document.getElementById('ajax-form').addEventListener('submit', function(e) {
    e.preventDefault();
    
    const form = this;
    const formData = new FormData(form);
    
    fetch(form.action, {
        method: 'POST',
        body: formData,
        headers: {
            'X-Requested-With': 'XMLHttpRequest',
            'X-CSRFToken': getCookie('csrftoken')
        }
    })
    .then(response => response.json())
    .then(data => {
        if (data.success) {
            window.location.href = data.redirect_url;
        } else {
            // Display errors
            Object.keys(data.field_errors).forEach(field => {
                const fieldElement = document.getElementById(`id_${field}`);
                const errorElement = document.getElementById(`${field}-error`);
                
                if (fieldElement) {
                    fieldElement.classList.add('is-invalid');
                }
                if (errorElement) {
                    errorElement.textContent = data.field_errors[field].join(', ');
                }
            });
        }
    })
    .catch(error => console.error('Error:', error));
});
```

---

## Best Practices

### 1. Form Organization
```python
# Keep forms in forms.py
# Use ModelForms when working with models
# Create separate forms for different use cases (create vs update)
```

### 2. Validation Best Practices
```python
class BestPracticeForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'content']
    
    def clean_title(self):
        # Clean, normalize, validate
        title = self.cleaned_data['title'].strip()
        if len(title) < 5:
            raise forms.ValidationError("Title too short")
        return title
    
    def clean(self):
        # Cross-field validation always in clean()
        cleaned_data = super().clean()
        # Always use .get() for potentially missing fields
        title = cleaned_data.get('title')
        content = cleaned_data.get('content')
        if title and content and title.lower() in content.lower():
            raise forms.ValidationError(
                "Content should not contain the title"
            )
        return cleaned_data
```

### 3. Security Considerations
```python
# Always use CSRF protection
# Validate all user input
# Use Django's built-in validators when possible
# Don't trust client-side validation alone
# Sanitize output in templates
```

### 4. Performance Tips
```python
# Use select_related/prefetch_related for ForeignKey/M2M fields in forms
class OrderForm(ModelForm):
    class Meta:
        model = Order
        fields = '__all__'
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        # Optimize querysets for dropdown fields
        self.fields['customer'].queryset = Customer.objects.select_related(
            'profile'
        )
        self.fields['products'].queryset = Product.objects.prefetch_related(
            'categories'
        )
```

### 5. Testing Forms
```python
from django.test import TestCase

class ContactFormTests(TestCase):
    def test_valid_form(self):
        data = {
            'name': 'John Doe',
            'email': 'john@example.com',
            'message': 'Test message'
        }
        form = ContactForm(data=data)
        self.assertTrue(form.is_valid())
    
    def test_invalid_form(self):
        data = {
            'name': '',  # Required field empty
            'email': 'invalid-email',
            'message': ''
        }
        form = ContactForm(data=data)
        self.assertFalse(form.is_valid())
        self.assertEqual(len(form.errors), 3)
    
    def test_clean_method(self):
        data = {
            'name': 'john',
            'email': 'john@example.com'
        }
        form = ContactForm(data=data)
        # Should fail because name is in email
        self.assertFalse(form.is_valid())
```
