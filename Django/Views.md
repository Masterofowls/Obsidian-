

## Function-Based Views (FBV)

### Basic Structure

```python
from django.shortcuts import render, redirect, get_object_or_404
from django.http import HttpResponse, JsonResponse, Http404
from django.contrib.auth.decorators import login_required
from django.views.decorators.http import require_http_methods
from .models import Post, Comment
from .forms import PostForm, CommentForm

# Simple view returning HTML
def home_view(request):
    return HttpResponse("<h1>Welcome to My Site</h1>")

# View with template rendering
def post_list(request):
    posts = Post.objects.filter(status='published').order_by('-created_at')
    context = {
        'posts': posts,
        'title': 'Latest Posts',
        'total_posts': posts.count()
    }
    return render(request, 'blog/post_list.html', context)

# View with URL parameters
def post_detail(request, pk):
    # post = Post.objects.get(pk=pk)  # Raises DoesNotExist if not found
    post = get_object_or_404(Post, pk=pk)  # Better: raises Http404
    return render(request, 'blog/post_detail.html', {'post': post})

# View with slug parameter
def post_by_slug(request, slug):
    post = get_object_or_404(Post, slug=slug, status='published')
    return render(request, 'blog/post_detail.html', {'post': post})
```

### Handling Different HTTP Methods

```python
@require_http_methods(["GET", "POST"])
def post_create(request):
    if request.method == 'POST':
        # Process form submission
        form = PostForm(request.POST, request.FILES)  # Include files
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.save()
            return redirect('post_detail', pk=post.pk)
    else:
        # Display empty form
        form = PostForm()
    
    return render(request, 'blog/post_form.html', {'form': form})

@require_http_methods(["GET", "POST", "DELETE"])
def post_update(request, pk):
    post = get_object_or_404(Post, pk=pk)
    
    if request.method == 'DELETE':
        post.delete()
        return redirect('post_list')
    
    if request.method == 'POST':
        form = PostForm(request.POST, request.FILES, instance=post)
        if form.is_valid():
            form.save()
            return redirect('post_detail', pk=post.pk)
    else:
        form = PostForm(instance=post)
    
    return render(request, 'blog/post_form.html', {'form': form, 'post': post})
```

### Using Decorators

```python
from django.contrib.auth.decorators import login_required, permission_required
from django.views.decorators.cache import cache_page
from django.views.decorators.csrf import csrf_exempt, csrf_protect
from django.views.decorators.gzip import gzip_page

@login_required  # Require authentication
def dashboard(request):
    return render(request, 'dashboard.html')

@login_required(login_url='/accounts/login/')  # Custom login URL
def profile(request):
    return render(request, 'profile.html')

@permission_required('blog.add_post', raise_exception=True)
def create_post(request):
    # Only users with add_post permission can access
    pass

@cache_page(60 * 15)  # Cache for 15 minutes
def cached_view(request):
    posts = Post.objects.all()  # Expensive query
    return render(request, 'blog/post_list.html', {'posts': posts})

@csrf_exempt  # Disable CSRF protection (use carefully!)
def webhook_receiver(request):
    # Process external webhook
    return HttpResponse("OK")

@require_http_methods(["GET"])  # Only GET requests
def read_only_view(request):
    return render(request, 'read_only.html')

@gzip_page  # Compress response
def large_content(request):
    return render(request, 'large_page.html')
```

### Returning Different Response Types

```python
def json_response_view(request):
    data = {'name': 'John', 'age': 30, 'city': 'New York'}
    return JsonResponse(data)
    # Also supports: JsonResponse(data, safe=False) for non-dict objects

def file_download(request, file_id):
    from django.http import FileResponse
    document = get_object_or_404(Document, pk=file_id)
    response = FileResponse(document.file.open(), as_attachment=True)
    response['Content-Disposition'] = f'attachment; filename="{document.filename}"'
    return response

def streaming_view(request):
    from django.http import StreamingHttpResponse
    import csv
    
    def generate_csv():
        yield ['Name', 'Email', 'Phone']
        for user in User.objects.all():
            yield [user.name, user.email, user.phone]
    
    response = StreamingHttpResponse(
        (','.join(row) + '\n' for row in generate_csv()),
        content_type='text/csv'
    )
    response['Content-Disposition'] = 'attachment; filename="users.csv"'
    return response

def custom_error_view(request):
    from django.http import HttpResponseNotFound, HttpResponseForbidden
    from django.http import HttpResponseServerError, HttpResponseBadRequest
    
    # Return different status codes
    if not request.user.is_authenticated:
        return HttpResponseForbidden("Access Denied")
    
    if some_condition:
        return HttpResponseBadRequest("Bad Request")
    
    if not found:
        return HttpResponseNotFound("Not Found")
    
    return HttpResponse("OK")
```

---

## Class-Based Views (CBV)

### Basic Class-Based View

```python
from django.views import View
from django.views.generic import TemplateView, RedirectView
from django.contrib.auth.mixins import LoginRequiredMixin

# Basic View class
class MyView(View):
    template_name = 'my_template.html'
    
    def get(self, request, *args, **kwargs):
        context = {'message': 'Hello from GET'}
        return render(request, self.template_name, context)
    
    def post(self, request, *args, **kwargs):
        # Handle POST request
        data = request.POST.get('data')
        return HttpResponse(f"Received: {data}")
    
    def dispatch(self, request, *args, **kwargs):
        # Method called for all HTTP methods - good for checks
        if not request.user.is_authenticated:
            return redirect('login')
        return super().dispatch(request, *args, **kwargs)
```

### TemplateView

```python
class HomeView(TemplateView):
    template_name = 'home.html'
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['latest_posts'] = Post.objects.all()[:5]
        context['popular_tags'] = Tag.objects.all()[:10]
        context['current_user'] = self.request.user
        return context

# Using TemplateView in URLs
# path('', HomeView.as_view(), name='home')
```

### RedirectView

```python
class OldURLRedirect(RedirectView):
    pattern_name = 'new_url'  # Redirect to named URL pattern
    permanent = True  # 301 Permanent Redirect (default: False = 302)
    query_string = True  # Pass query string to new URL
    
    def get_redirect_url(self, *args, **kwargs):
        # Dynamic redirect based on logic
        post = get_object_or_404(Post, pk=kwargs['pk'])
        return post.get_absolute_url()

class ExternalRedirect(RedirectView):
    url = 'https://example.com'  # Hard-coded URL
```

---

## Generic Display Views

### ListView

```python
from django.views.generic import ListView
from django.db.models import Q, Count

class PostListView(ListView):
    model = Post
    template_name = 'blog/post_list.html'
    context_object_name = 'posts'
    paginate_by = 10  # Paginate: 10 items per page
    ordering = ['-created_at']
    
    def get_queryset(self):
        # Custom queryset filtering
        queryset = Post.objects.filter(status='published')
        
        # Search functionality
        search_query = self.request.GET.get('q')
        if search_query:
            queryset = queryset.filter(
                Q(title__icontains=search_query) |
                Q(content__icontains=search_query)
            )
        
        # Category filter
        category = self.kwargs.get('category')
        if category:
            queryset = queryset.filter(category__slug=category)
        
        return queryset
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['categories'] = Category.objects.all()
        context['popular_posts'] = Post.objects.annotate(
            comment_count=Count('comments')
        ).order_by('-comment_count')[:5]
        return context

# Multiple model ListView
class SearchResultsView(ListView):
    template_name = 'search_results.html'
    context_object_name = 'results'
    
    def get_queryset(self):
        query = self.request.GET.get('q')
        post_results = Post.objects.filter(title__icontains=query)
        user_results = User.objects.filter(username__icontains=query)
        # Combine different querysets
        return {'posts': post_results, 'users': user_results}
```

### DetailView

```python
from django.views.generic import DetailView

class PostDetailView(DetailView):
    model = Post
    template_name = 'blog/post_detail.html'
    context_object_name = 'post'
    slug_field = 'slug'
    slug_url_kwarg = 'slug'
    pk_url_kwarg = 'pk'
    query_pk_and_slug = False  # Look up by both PK and slug
    
    def get_object(self, queryset=None):
        # Custom object retrieval
        obj = super().get_object(queryset=queryset)
        # Increment view count
        obj.views += 1
        obj.save(update_fields=['views'])
        return obj
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['comments'] = self.object.comments.filter(approved=True)
        context['related_posts'] = Post.objects.filter(
            category=self.object.category
        ).exclude(pk=self.object.pk)[:5]
        context['comment_form'] = CommentForm()
        return context
    
    def get_queryset(self):
        # Restrict access to published posts only
        if self.request.user.is_staff:
            return Post.objects.all()
        return Post.objects.filter(status='published')

# Single Object with Custom Slug
class ProductDetailView(DetailView):
    model = Product
    slug_field = 'product_code'  # Custom slug field
    slug_url_kwarg = 'code'      # URL parameter name
```

### Archive Views (Date-Based)

```python
from django.views.generic.dates import (
    ArchiveIndexView, YearArchiveView, MonthArchiveView,
    WeekArchiveView, DayArchiveView, TodayArchiveView
)

class PostArchiveView(ArchiveIndexView):
    model = Post
    date_field = 'published_date'  # Required
    template_name = 'blog/post_archive.html'
    allow_empty = True  # Show page even if no posts
    allow_future = False  # Don't show future posts
    paginate_by = 10
    context_object_name = 'posts'

class PostYearArchiveView(YearArchiveView):
    model = Post
    date_field = 'published_date'
    make_object_list = True  # Get full object list, not just months
    year_format = '%Y'
    template_name = 'blog/post_year_archive.html'

class PostMonthArchiveView(MonthArchiveView):
    model = Post
    date_field = 'published_date'
    month_format = '%m'
    template_name = 'blog/post_month_archive.html'

class PostWeekArchiveView(WeekArchiveView):
    model = Post
    date_field = 'published_date'
    week_format = '%W'  # ISO week number
    template_name = 'blog/post_week_archive.html'

class PostDayArchiveView(DayArchiveView):
    model = Post
    date_field = 'published_date'
    month_format = '%m'
    day_format = '%d'
    template_name = 'blog/post_day_archive.html'

class PostTodayArchiveView(TodayArchiveView):
    model = Post
    date_field = 'published_date'
    template_name = 'blog/post_today_archive.html'
```

---

## Generic Editing Views

### CreateView

```python
from django.views.generic.edit import CreateView
from django.urls import reverse_lazy

class PostCreateView(LoginRequiredMixin, CreateView):
    model = Post
    form_class = PostForm
    template_name = 'blog/post_form.html'
    success_url = reverse_lazy('post_list')  # Redirect after creation
    # Or use: success_url = '/' 
    
    def form_valid(self, form):
        # Set the author before saving
        form.instance.author = self.request.user
        # Add any custom processing
        response = super().form_valid(form)
        # Send notification, etc.
        return response
    
    def form_invalid(self, form):
        # Log errors or add custom handling
        return super().form_invalid(form)
    
    def get_form_kwargs(self):
        # Pass additional arguments to form
        kwargs = super().get_form_kwargs()
        kwargs['user'] = self.request.user
        return kwargs
    
    def get_initial(self):
        # Initial form values
        initial = super().get_initial()
        initial['author'] = self.request.user
        initial['status'] = 'draft'
        return initial
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['title'] = 'Create New Post'
        context['submit_text'] = 'Create'
        return context
```

### UpdateView

```python
from django.views.generic.edit import UpdateView

class PostUpdateView(LoginRequiredMixin, UpdateView):
    model = Post
    form_class = PostForm
    template_name = 'blog/post_form.html'
    # success_url = reverse_lazy('post_list')
    
    def get_success_url(self):
        # Dynamic success URL based on post
        return reverse_lazy('post_detail', kwargs={'pk': self.object.pk})
    
    def get_object(self, queryset=None):
        obj = super().get_object(queryset=queryset)
        # Check ownership
        if obj.author != self.request.user and not self.request.user.is_staff:
            raise PermissionDenied("You can't edit this post")
        return obj
    
    def get_queryset(self):
        # Only allow editing own posts
        return Post.objects.filter(author=self.request.user)
    
    def form_valid(self, form):
        # Custom validation or processing
        if form.cleaned_data['status'] == 'published' and not self.object.was_published:
            form.instance.published_date = timezone.now()
        return super().form_valid(form)

# Update with inline formsets
class OrderUpdateView(UpdateView):
    model = Order
    form_class = OrderForm
    template_name = 'orders/order_form.html'
    success_url = reverse_lazy('order_list')
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        if self.request.POST:
            context['order_items'] = OrderItemFormSet(
                self.request.POST, instance=self.object
            )
        else:
            context['order_items'] = OrderItemFormSet(instance=self.object)
        return context
    
    def form_valid(self, form):
        context = self.get_context_data()
        order_items = context['order_items']
        if order_items.is_valid():
            self.object = form.save()
            order_items.instance = self.object
            order_items.save()
            return super().form_valid(form)
        return self.render_to_response(self.get_context_data(form=form))
```

### DeleteView

```python
from django.views.generic.edit import DeleteView

class PostDeleteView(LoginRequiredMixin, DeleteView):
    model = Post
    template_name = 'blog/post_confirm_delete.html'
    success_url = reverse_lazy('post_list')
    context_object_name = 'post'
    
    def get_queryset(self):
        # Only allow deleting own posts
        return Post.objects.filter(author=self.request.user)
    
    def delete(self, request, *args, **kwargs):
        # Add logging or processing before delete
        self.object = self.get_object()
        success_url = self.get_success_url()
        # Log deletion
        logger.info(f"Post {self.object.pk} deleted by {request.user}")
        self.object.delete()
        return HttpResponseRedirect(success_url)

# Soft delete implementation
class PostSoftDeleteView(LoginRequiredMixin, UpdateView):
    model = Post
    fields = []  # No fields to edit
    template_name = 'blog/post_confirm_delete.html'
    
    def form_valid(self, form):
        # Soft delete: set is_deleted flag
        self.object.is_deleted = True
        self.object.save()
        return redirect('post_list')
```

### FormView

```python
from django.views.generic.edit import FormView

class ContactFormView(FormView):
    template_name = 'contact.html'
    form_class = ContactForm
    success_url = reverse_lazy('contact_success')
    
    def form_valid(self, form):
        # Process the form data
        form.send_email()
        return super().form_valid(form)
    
    def get_form_kwargs(self):
        kwargs = super().get_form_kwargs()
        kwargs['request'] = self.request
        return kwargs
    
    def get_initial(self):
        initial = super().get_initial()
        if self.request.user.is_authenticated:
            initial['name'] = self.request.user.get_full_name()
            initial['email'] = self.request.user.email
        return initial
```

---

## Authentication Views

### Built-in Auth Views

```python
from django.contrib.auth.views import (
    LoginView, LogoutView, PasswordChangeView,
    PasswordChangeDoneView, PasswordResetView,
    PasswordResetDoneView, PasswordResetConfirmView,
    PasswordResetCompleteView
)

# Custom Login View
class CustomLoginView(LoginView):
    template_name = 'registration/login.html'
    redirect_authenticated_user = True  # Redirect if already logged in
    next_page = reverse_lazy('dashboard')
    
    def get_success_url(self):
        url = self.get_redirect_url()
        return url or reverse_lazy('dashboard')
    
    def form_valid(self, form):
        # Add custom logic (logging, etc.)
        remember_me = form.cleaned_data.get('remember_me')
        if not remember_me:
            self.request.session.set_expiry(0)  # Session expires on browser close
        return super().form_valid(form)

# Custom Logout View
class CustomLogoutView(LogoutView):
    next_page = reverse_lazy('login')
    
    def dispatch(self, request, *args, **kwargs):
        # Custom logout logic
        if request.user.is_authenticated:
            # Log logout event
            logger.info(f"User {request.user} logged out")
        return super().dispatch(request, *args, **kwargs)

# Password Change
class CustomPasswordChangeView(PasswordChangeView):
    template_name = 'registration/password_change.html'
    success_url = reverse_lazy('password_change_done')
    
# Password Reset
class CustomPasswordResetView(PasswordResetView):
    template_name = 'registration/password_reset.html'
    email_template_name = 'registration/password_reset_email.html'
    subject_template_name = 'registration/password_reset_subject.txt'
    success_url = reverse_lazy('password_reset_done')
```

---

## Advanced View Concepts

### Mixins (Class-Based View Composition)

```python
from django.contrib.auth.mixins import (
    LoginRequiredMixin, PermissionRequiredMixin,
    UserPassesTestMixin, AccessMixin
)
from django.core.exceptions import PermissionDenied

# Custom Mixins
class AuthorRequiredMixin:
    """Ensure only the author can access the view"""
    def dispatch(self, request, *args, **kwargs):
        obj = self.get_object()
        if obj.author != request.user:
            raise PermissionDenied("You are not the author")
        return super().dispatch(request, *args, **kwargs)

class StaffRequiredMixin(LoginRequiredMixin):
    """Ensure user is staff member"""
    def dispatch(self, request, *args, **kwargs):
        if not request.user.is_staff:
            return self.handle_no_permission()
        return super().dispatch(request, *args, **kwargs)

class AJAXRequiredMixin:
    """Handle AJAX requests only"""
    def dispatch(self, request, *args, **kwargs):
        if not request.headers.get('x-requested-with') == 'XMLHttpRequest':
            return HttpResponseBadRequest('AJAX requests only')
        return super().dispatch(request, *args, **kwargs)

class FormValidMessageMixin:
    """Add success message after form submission"""
    success_message = "Changes saved successfully"
    
    def form_valid(self, form):
        response = super().form_valid(form)
        messages.success(self.request, self.success_message)
        return response

# Using multiple mixins
class AdminEditView(
    LoginRequiredMixin,
    PermissionRequiredMixin,
    FormValidMessageMixin,
    UpdateView
):
    model = Post
    form_class = PostForm
    template_name = 'admin/edit_post.html'
    permission_required = 'blog.change_post'
    success_message = "Post updated successfully"
    success_url = reverse_lazy('admin_dashboard')
```

### AJAX Views

```python
from django.views.decorators.http import require_POST

# Function-based AJAX view
@require_POST
@login_required
def like_post_ajax(request, post_id):
    post = get_object_or_404(Post, pk=post_id)
    
    if request.user in post.likes.all():
        post.likes.remove(request.user)
        liked = False
    else:
        post.likes.add(request.user)
        liked = True
    
    return JsonResponse({
        'liked': liked,
        'count': post.likes.count(),
        'status': 'success'
    })

# Class-based AJAX view
class AJAXPostLikeView(LoginRequiredMixin, View):
    def post(self, request, *args, **kwargs):
        post = get_object_or_404(Post, pk=kwargs['pk'])
        
        data = json.loads(request.body)
        action = data.get('action', 'like')
        
        if action == 'like':
            post.likes.add(request.user)
        elif action == 'unlike':
            post.likes.remove(request.user)
        
        return JsonResponse({
            'success': True,
            'likes_count': post.likes.count(),
            'is_liked': request.user in post.likes.all()
        })

# Infinite scroll implementation
class InfiniteScrollListView(ListView):
    model = Post
    template_name = 'blog/post_list_infinite.html'
    paginate_by = 10
    
    def get_template_names(self):
        if self.request.headers.get('x-requested-with') == 'XMLHttpRequest':
            return ['blog/post_list_ajax.html']  # AJAX template
        return [self.template_name]
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        if self.request.headers.get('x-requested-with') == 'XMLHttpRequest':
            context['is_ajax'] = True
        return context
```

### File Handling Views

```python
class FileUploadView(FormView):
    template_name = 'upload.html'
    form_class = FileUploadForm
    success_url = reverse_lazy('upload_success')
    
    def form_valid(self, form):
        files = self.request.FILES.getlist('files')
        for file in files:
            # Process each file
            Document.objects.create(
                file=file,
                name=file.name,
                uploaded_by=self.request.user
            )
        return super().form_valid(form)

class MultipleFileUploadView(View):
    def post(self, request, *args, **kwargs):
        files = request.FILES.getlist('files')
        uploaded = []
        
        for file in files:
            # Validate file type and size
            if not file.content_type in ['image/jpeg', 'image/png']:
                continue
            if file.size > 5 * 1024 * 1024:  # 5MB limit
                continue
            
            # Save file
            instance = Photo.objects.create(
                image=file,
                uploaded_by=request.user
            )
            uploaded.append({
                'id': instance.id,
                'url': instance.image.url,
                'name': file.name
            })
        
        return JsonResponse({'files': uploaded})

class FileDownloadView(View):
    def get(self, request, file_id):
        document = get_object_or_404(Document, pk=file_id)
        file_path = document.file.path
        
        response = FileResponse(
            open(file_path, 'rb'),
            as_attachment=True,
            filename=document.original_filename
        )
        
        # Set content type
        response['Content-Type'] = document.content_type
        return response
```

### API Views

```python
from django.core.serializers import serialize

# Simple API view
def api_post_list(request):
    if request.method == 'GET':
        posts = Post.objects.filter(status='published')
        data = []
        for post in posts:
            data.append({
                'id': post.id,
                'title': post.title,
                'content': post.content,
                'author': post.author.username,
                'created_at': post.created_at.isoformat(),
                'url': post.get_absolute_url()
            })
        return JsonResponse({'posts': data})
    
    elif request.method == 'POST':
        data = json.loads(request.body)
        post = Post.objects.create(
            title=data['title'],
            content=data['content'],
            author=request.user
        )
        return JsonResponse({
            'id': post.id,
            'status': 'created'
        }, status=201)

# REST-like class-based API view
class PostAPIView(View):
    def get(self, request, post_id=None):
        if post_id:
            post = get_object_or_404(Post, pk=post_id)
            data = {
                'id': post.id,
                'title': post.title,
                'content': post.content
            }
        else:
            posts = Post.objects.all()
            data = [
                {'id': p.id, 'title': p.title}
                for p in posts
            ]
        return JsonResponse(data, safe=False)
    
    def post(self, request):
        data = json.loads(request.body)
        post = Post.objects.create(**data)
        return JsonResponse({'id': post.id}, status=201)
    
    def put(self, request, post_id):
        data = json.loads(request.body)
        post = get_object_or_404(Post, pk=post_id)
        for key, value in data.items():
            setattr(post, key, value)
        post.save()
        return JsonResponse({'status': 'updated'})
    
    def delete(self, request, post_id):
        post = get_object_or_404(Post, pk=post_id)
        post.delete()
        return JsonResponse({'status': 'deleted'}, status=204)
```

---

## URL Configuration

```python
# urls.py
from django.urls import path, include, re_path
from . import views

urlpatterns = [
    # Function-based views
    path('', views.home_view, name='home'),
    path('posts/', views.post_list, name='post_list'),
    path('posts/<int:pk>/', views.post_detail, name='post_detail'),
    path('posts/<slug:slug>/', views.post_by_slug, name='post_by_slug'),
    
    # Class-based views
    path('about/', TemplateView.as_view(
        template_name='about.html',
        extra_context={'title': 'About Us'}
    ), name='about'),
    
    path('posts/', PostListView.as_view(), name='post_list'),
    path('post/<int:pk>/', PostDetailView.as_view(), name='post_detail'),
    path('post/create/', PostCreateView.as_view(), name='post_create'),
    path('post/<int:pk>/edit/', PostUpdateView.as_view(), name='post_update'),
    path('post/<int:pk>/delete/', PostDeleteView.as_view(), name='post_delete'),
    
    # Authentication URLs
    path('accounts/', include('django.contrib.auth.urls')),
    path('accounts/login/', CustomLoginView.as_view(), name='login'),
    path('accounts/logout/', CustomLogoutView.as_view(), name='logout'),
    
    # API URLs
    path('api/posts/', PostAPIView.as_view(), name='api_posts'),
    path('api/posts/<int:post_id>/', PostAPIView.as_view(), name='api_post_detail'),
    
    # Redirects
    path('old-path/', RedirectView.as_view(
        url='/new-path/', permanent=True
    )),
]
```

---

## View Testing

```python
from django.test import TestCase, Client
from django.urls import reverse
from django.contrib.auth.models import User

class ViewTests(TestCase):
    def setUp(self):
        self.client = Client()
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )
        self.post = Post.objects.create(
            title='Test Post',
            content='Test Content',
            author=self.user
        )
    
    def test_post_list_view(self):
        response = self.client.get(reverse('post_list'))
        self.assertEqual(response.status_code, 200)
        self.assertTemplateUsed(response, 'blog/post_list.html')
        self.assertContains(response, 'Test Post')
    
    def test_post_detail_view(self):
        response = self.client.get(
            reverse('post_detail', kwargs={'pk': self.post.pk})
        )
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.context['post'], self.post)
    
    def test_create_post_authenticated(self):
        self.client.login(username='testuser', password='testpass123')
        response = self.client.post(reverse('post_create'), {
            'title': 'New Post',
            'content': 'New Content',
            'status': 'draft'
        })
        self.assertEqual(response.status_code, 302)  # Redirect after creation
        self.assertEqual(Post.objects.count(), 2)
    
    def test_create_post_unauthenticated(self):
        response = self.client.post(reverse('post_create'), {
            'title': 'New Post',
            'content': 'New Content'
        })
        self.assertEqual(response.status_code, 302)  # Redirect to login
        self.assertTrue('/accounts/login/' in response.url)
```

---

## Best Practices

### 1. Keep Views Thin
```python
# Bad: Business logic in view
def post_create(request):
    if request.method == 'POST':
        # Lots of business logic here
        post = Post()
        post.title = request.POST['title']
        post.slug = slugify(post.title)
        # ... 50 lines of logic
    
# Good: Move logic to models/services
def post_create(request):
    if request.method == 'POST':
        form = PostForm(request.POST)
        if form.is_valid():
            post_service = PostService()
            post = post_service.create_post(
                author=request.user,
                **form.cleaned_data
            )
            return redirect('post_detail', pk=post.pk)
```

### 2. Use Class-Based Views When Appropriate
```python
# Use CBV for standard CRUD operations
# Use FBV for complex, non-standard logic
```

### 3. Handle Permissions Properly
```python
class SecureView(LoginRequiredMixin, UserPassesTestMixin, View):
    def test_func(self):
        return self.request.user.has_perm('app.permission')
    
    def handle_no_permission(self):
        if self.request.user.is_authenticated:
            raise PermissionDenied
        return super().handle_no_permission()
```

### 4. Optimize Queries
```python
class PostListView(ListView):
    def get_queryset(self):
        # Use select_related for ForeignKey
        # Use prefetch_related for ManyToMany
        return Post.objects.select_related('author').prefetch_related('tags')
```

### 5. Use Proper HTTP Status Codes
```python
def api_view(request):
    if not request.user.is_authenticated:
        return JsonResponse({'error': 'Unauthorized'}, status=401)
    if not valid:
        return JsonResponse({'error': 'Bad Request'}, status=400)
    if created:
        return JsonResponse(data, status=201)
    return JsonResponse(data, status=200)
```

### 6. Add Caching
```python
from django.views.decorators.cache import cache_page
from django.utils.decorators import method_decorator

@method_decorator(cache_page(60 * 15), name='dispatch')
class CachedListView(ListView):
    pass
```

### 7. Handle Errors Gracefully
```python
class MyView(View):
    def dispatch(self, request, *args, **kwargs):
        try:
            return super().dispatch(request, *args, **kwargs)
        except ObjectDoesNotExist:
            raise Http404("Object not found")
        except PermissionDenied:
            raise
        except Exception as e:
            logger.error(f"Error in {self.__class__.__name__}: {e}")
            raise
```

