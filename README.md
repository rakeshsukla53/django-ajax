Django Ajax Calls Tutorial

**Django Login Middleware**
 
 Text and code take from [Login Middleware](http://onecreativeblog.com/post/59051248/django-login-required-middleware)
 
 I’ve been doing a lot programming in the Django web framework lately and thought this code snippet that forces a user to login to your site before viewing any pages might come in handy. If you’re already a djangoperson, skip ahead to the code, otherwise, I’ll give a bit of background.

In Django, your code that responds to each incoming request is organized into functions called views which take your HTTP request and turn it into a page that the user can view. It’s a common practice in Django to augment your views using a python construct called a decorator. This allows you add additional functionality without having to write a bunch of code. For example, Django provides a decorator called login_required that takes one of your views and prevents a user from seeing it unless they are logged in:

    @login_required      # This is the decorator! One line... simple.
    def my_view(request):
        # Do something here to turn the request into an HTML page.
        # ...

Always use the middleware if you want all the user to go through this filter before coming to view the page.

Sometimes it’s a real pain to use the login_required decorator all over the views of your complicated site. What if you forget to add it to view that contains sensitive information? Fortunately, Django allows you to write middleware that gets access to each request so you can add functionality that can be applied to your whole site. My middleware simply intercepts each request and redirects users to the site login page if they haven’t logged in. It also allows you to give of exceptions (in the form of regular expressions), i.e. pages that can be viewed without logging in:

    from django.http import HttpResponseRedirect
    from django.conf import settings
    from re import compile
    
    EXEMPT_URLS = [compile(settings.LOGIN_URL.lstrip('/'))]
    if hasattr(settings, 'LOGIN_EXEMPT_URLS'):
        EXEMPT_URLS += [compile(expr) for expr in settings.LOGIN_EXEMPT_URLS]
    
    class LoginRequiredMiddleware:
        """
        Middleware that requires a user to be authenticated to view any page other
        than LOGIN_URL. Exemptions to this requirement can optionally be specified
        in settings via a list of regular expressions in LOGIN_EXEMPT_URLS (which
        you can copy from your urls.py).
    
        Requires authentication middleware and template context processors to be
        loaded. You'll get an error if they aren't.
        """
        def process_request(self, request):
            assert hasattr(request, 'user'), "The Login Required middleware\
     requires authentication middleware to be installed. Edit your\
     MIDDLEWARE_CLASSES setting to insert\
     'django.contrib.auth.middlware.AuthenticationMiddleware'. If that doesn't\
     work, ensure your TEMPLATE_CONTEXT_PROCESSORS setting includes\
     'django.core.context_processors.auth'."
            if not request.user.is_authenticated():
                path = request.path_info.lstrip('/')
                if not any(m.match(path) for m in EXEMPT_URLS):
                    return HttpResponseRedirect(settings.LOGIN_URL)
    
    from django.http import HttpResponseRedirect
    from django.conf import settings
    from re import compile
    
    EXEMPT_URLS = [compile(settings.LOGIN_URL.lstrip('/'))]
    if hasattr(settings, 'LOGIN_EXEMPT_URLS'):
        EXEMPT_URLS += [compile(expr) for expr in settings.LOGIN_EXEMPT_URLS]
    
    class LoginRequiredMiddleware:
        """
        Middleware that requires a user to be authenticated to view any page other
        than LOGIN_URL. Exemptions to this requirement can optionally be specified
        in settings via a list of regular expressions in LOGIN_EXEMPT_URLS (which
        you can copy from your urls.py).
        Requires authentication middleware and template context processors to be
        loaded. You'll get an error if they aren't.
        """
        def process_request(self, request):
            assert hasattr(request, 'user'), "The Login Required middleware\
     requires authentication middleware to be installed. Edit your\
     MIDDLEWARE_CLASSES setting to insert\
     'django.contrib.auth.middleware.AuthenticationMiddleware'. If that doesn't\
     work, ensure your TEMPLATE_CONTEXT_PROCESSORS setting includes\
     'django.core.context_processors.auth'."
            if not request.user.is_authenticated():
                path = request.path_info.lstrip('/')
                if not any(m.match(path) for m in EXEMPT_URLS):
                    return HttpResponseRedirect(settings.LOGIN_URL)
    view raw
    login_required_middleware.py hosted with ❤ by GitHub
    
    Example:
    
    # settings.py
    
    LOGIN_URL = '/login/'
    
    LOGIN_EXEMPT_URLS = (
     r'^about\.html$',
     r'^legal/', # allow any URL under /legal/*
    ) 
    
    MIDDLEWARE_CLASSES = (
        # ...
        'python.path.to.LoginRequiredMiddleware',
    )
    
    # settings.py
    
    LOGIN_URL = '/login/'
    
    LOGIN_EXEMPT_URLS = (
     r'^about\.html$',
     r'^legal/', # allow any URL under /legal/*
    ) 
    
    MIDDLEWARE_CLASSES = (
        # ...
        'python.path.to.LoginRequiredMiddleware',
    )
    view raw
    gistfile1.py hosted with ❤ by GitHub
    
    While writing this, I found similar solutions by davyd and a discussion on the django-users list, but neither had the flexibility of using regular expressions.
 










