# Practical Django Projects

Just some extracts from the "Practical Django Projects" book which capture my attention

## Data types

### strings

Django will represent the URL, title, and tags as Unicode strings. Ordinarily, Django’s
practice of ensuring that strings stored in model fields are Unicode is a good thing: it removes
a lot of the headaches of dealing with character encodings. But in this case, it’s slightly prob-
lematic as well because Unicode strings don’t translate directly into a series of binary bytes, so
they aren’t suitable to be sent out “over the wire” in a web-based API call.
So you’ll need to convert the values of these fields into byte-based strings before passing
them to the Delicious API. Django provides a helper function, `django.utils.encoding.smart_str()`, which will do this. In a lot of cases, you could probably also use Python’s built-in `str()`
function and get away with it. However, Django’s `smart_str()` can handle some situations that
`str()` can’t, and it also defaults to encoding the result in UTF-8 instead of ASCII (which is the
default for most Python installations).
Django uses Unicode strings everywhere, so whenever you use an
external API, you should convert Unicode strings to bytestrings by using the helper function
`django.utils.encoding.smart_str()`.

Read http://www.rrn.dk/the-difference-between-utf-8-and-unicode

### datetime

    delta = datetime.datetime.now() - entry.pub_date
    if delta.days > 30:
        instance.is_public = False

the delta var is an instance of a class called timedelta

## Imports
Your new tag, however, is going to get an argument like coltrane.link or coltrane.
entry, so you will need to import the correct model class dynamically.
Python provides a way to do this through a special built-in function named `__import__()`,
which takes strings as arguments. But loading a model class dynamically is a common enough
need that Django provides a helper function to handle it more concisely. This function is
`django.db.models.get_model()`, and it takes two arguments:

- The name of the application the model is defined in, as a string
- The name of the model class, as a string

## Classes
So you can start by writing its constructor (remember that a Python object’s constructor is
always called `__init__()`) and simply storing those arguments as instance variables:

    class LatestContentNode(template.Node):
        def __init__(self, model, num, varname):
        self.model = model
        self.num = int(num)
        self.varname = varname

### inheritance
So far, you’ve been writing models that are subclasses of Django’s built-in basic model
class, but Django also supports models that subclass from other model classes. It allows you to
use either of two common patterns when you’re doing such subclassing:

- Concrete inheritance: This is what many people think of when they imagine how subclassing a model works. In this pattern, one model that subclasses another will create a new database table that links back to the original “parent” class’s table with a foreign key. Instances of the subclassed model will behave as if they have both the fields defined on the “parent” model and the fields defined on the subclassed model itself (under the hood, Django will pull information from both tables as needed).
- Abstract inheritance: When you define a new model class and fill in its options using the inner class Meta declaration, you can add the attribute abstract=True. When you do this, Django will not create a table for that model, and you won’t be able to directly create or query for instances of that model. However, any subclasses of that model (as long as they don’t also declare abstract=True) will create their own tables, and will add columns for the fields from the abstract model as well.

Using this technique—accepting `*args` and `**kwargs` and passing them on to the parent method—is a useful shorthand when the method you’re overriding accepts a lot of arguments,
especially if a lot of them are optional.

## Functions & Methods

the asterisk (`*`) is special Python syntax for taking a list (the result of calling split()) and turning in a set of arguments to a function

this

    model_args = bits[1].split('.')
    model = get_model(*model_args)

is equal to

    model_args = bits[1].split('.')
    model = get_model(model_args[0], model_args[1])

## Settings

How to deal with local and production settings? Simple: Just have your settings.py file keep all the production settings, and then at the end of the file:

    try:
        from local_settings import *
    except ImportError:
        pass

Then add a `local_settings.py` file in the same directory where you override all the settings you need. In production this file won't exist, so will not be processed.

## Models

### Meta
The `abstract` options let's you define the class is an abstract one, so no tables are created, and no instances can be created. This is useful or models inheritance.
In other words, concrete inheritance creates one table for each model, as usual. Abstract inheritance creates only one table, the table for the subclass, and places all of the fields inside it.

### blank and null
Also, it’s important to note that for text-based field types (CharField, TextField, and others), Django
will never insert a NULL. For these field types, a blank value will be inserted as an empty string. This is to
avoid a situation where there are potentially two different blank values for the field (either an empty string or
a NULL) and to ensure that code that checks for blank values can be kept simple. Because of this, you should
generally avoid specifying null=True on text-based field types.

### related_name

    class Bookmark(models.Model):
        snippet = models.ForeignKey(Snippet)
        user = models.ForeignKey(User, related_name='cab_bookmarks')
        date = models.DateTimeField(editable=False)

The fact that you’ve created a foreign key to User means that Django will add a new attribute to every User object, which you’ll be able to use to access each user’s bookmarks. By default, this attribute would be named bookmark_set based on the name of your Bookmark model.
However, this can create a problem: If you ever use any other application with a bookmarking system, and if that application names its model Bookmark, you’ll get a naming conflict because the bookmark_set attribute of a User can’t simultaneously refer to two different models. The solution to this is the related_name argument to ForeignKey, which lets you manually specify the name of the new attribute on User, which you’ll use to access bookmarks. In this case, you’ll use the name cab_bookmarks. So once this model is installed and you have some bookmarks in your database, you’ll be able to run queries like this:

    from django.contrib.auth.models import User
    u = User.objects.get(pk=1)
    bookmarks = u.cab_bookmarks.all()

### unique keys
Most good weblog software
builds URLs that include the publication dates of entries (so that they look like /2007/10/09/
entry-title/), which means that all you really need is for the combination of the slug and the
publication date to be unique. Django provides an easy way to specify this, through an option
called unique_for_date:

    slug = models.SlugField(unique_for_date='pub_date')

### default

    pub_date = models.DateTimeField(default=datetime.datetime.now)

Notice that there aren’t any parentheses there—it’s datetime.datetime.now, not datetime.
datetime.now(). When you’re specifying a default, Django lets you supply either an appropri-
ate value or a function, which will generate the appropriate value on demand. In this case,
you’re supplying a function, and Django will call it whenever the default value is needed. This
ensures that the correct current datetime is generated each time.

### choices

    class Entry(models.Model):
        LIVE_STATUS = 1
        DRAFT_STATUS = 2
        STATUS_CHOICES = (
            (LIVE_STATUS, 'Live'),
            (DRAFT_STATUS, 'Draft'),
        )

Now instead of hard-coding the integer values anywhere you’re doing queries for specific
types of entries, you can instead refer to Entry.LIVE_STATUS or Entry.DRAFT_STATUS and know
that it’ll be the right value.

### generic relations
Generic relations actually involve two special field types, GenericForeignKey and
GenericRelation, that allow one model to have relationships with any other model installed in
your project.

see http://axiacore.com/blog/how-use-genericforeignkey-django/

### model class guidelines

- Any constants and/or lists of choices
- The full list of fields
- The `Meta` class, if present
- The `__unicode__()` method
- The `save()` method, if it’s being overridden
- The `get_absolute_url()` method, if present
- Any additional custom methods

### permalink
The permalink decorator you’re using here (which lives in django.db.models) will actually rewrite `the get_absolute_url()` function to do a reverse URL lookup. It will scan the project’s URLConf to look for the URL pattern with the specified name, then use that pattern’s regular expression to create the correct URL string and fill in the proper values for any arguments that need to be embedded in the URL.

This

    def get_absolute_url(self):
        return ('coltrane_entry_detail', (), { 'year': self.pub_date.strftime("%Y"),
            'month': self.pub_date.strftime("%b").lower(),
            'day': self.pub_date.strftime("%d"),
            'slug': self.slug })
    get_absolute_url = models.permalink(get_absolute_url)

is the same as this

    @models.permalink
    def get_absolute_url(self):
        return ('coltrane_entry_detail', (), { 'year': self.pub_date.strftime("%Y"),
            'month': self.pub_date.strftime("%b").lower(),
            'day': self.pub_date.strftime("%d"),
            'slug': self.slug })

###URLField
You won’t be able to enter a nonexistent or “broken” URL. By default, Django will issue an HTTP request to the URL during validation and will refuse to accept the URL if it returns an HTTP error status (such as “404 Not Found” or “500 Internal Server Error”). You can disable this verification by using the keyword argument `verify_exists=False` when setting up the `URLField`.

## Settings
You can access your Django settings file the same way you would access any other Python module—by importing it from its location on your computer (using import cms.settings, for example). However, it’s generally a better idea to use `from django.conf import settings`. This will enable a feature in Django that automatically supplies default values for many settings if you haven’t filled them in.

## Urls
Sometimes can be useful to split urls declarations in many files, in order to keep the code cleaner and easier to read and search for. For example in case you use something like this:

    (r'^weblog/', include('blog.urls')),

in the main URLconf, and then thousands of lines in the blog.urls file (categories index, entry index, entry detail, last entries, links, ...), you'd rather split this way:

    (r'^weblog/categories/', include('blog.urls.categories')),
    (r'^weblog/links/', include('blog.urls.links')),
    (r'^weblog/tags/', include('blog.urls.tags')),
    (r'^weblog/', include('blog.urls.entries')),

where the thousands of url are splitted into different files creawted inside an url directory inside the blog app (Add the `__ini__.py` file in the url dir!).
Because they’re no longer jumbled together into one file, it’s easy to use include() to put a specific group of patterns at any spot in a site’s URL hierarchy. This means you’re no longer tied to specific prefixes such as “links/” or “tags/” if you don’t want them.

## Managers

### why are they important?

It would be much nicer if you could have a separate way of querying entries that returns only objects with the status field set to Live, maybe something like `Entry.live.all()` instead of `Entry.objects.all(status=Entry.LIVE)`. This is actually pretty easy to do, but it requires the introduction of one more major feature of Django’s model system: managers.

### the `objects` attribute of models
The objects attribute is an instance of a special class (`django.db.models.Manager`), which is
meant to be “attached” to a particular model class, and which knows how to perform all sorts of
database queries.

In addition to the methods you’ve already seen—`all()` and `filter()`—it has
a large number of other methods that can return single specific objects, return lists of objects,
return other data structures corresponding to data stored by a model, change the ordering used
to return results, and handle a variety of other useful tasks

If you don’t specify a manager for your model, Django adds one and calls it objects (this
happens automatically for any class that subclasses `django.db.models.Model`). However, you’re
free to attach a manager with any name you like, and if you do, Django won’t bother with the
automatic default objects manager. For example, you could define a model like this:

    class MyModel(models.Model):
        name = models.CharField(max_length=50)

        object_fetcher = models.Manager()

Then instead of using `MyModel.objects.all()`, for example, you would use `MyModel.object_fetcher.all()`. All of the standard querying methods will be there, just in an attribute
with the name you’ve specified.

In this case, you want to write a manager that, when attached to the Entry model, will return only entries whose
status is Live. You can do this by writing a subclass of Manager and overriding one method,
`get_query_set()`, which returns the initial QuerySet object that all(), filter(), and all the
other querying methods will use. Doing this is surprisingly easy:

    class LiveEntryManager(models.Manager):
        def get_query_set(self):
            # self.model refers to the model the manager is attached to
            return super(LiveEntryManager, self).get_query_set().filter(status=self.model.LIVE_STATUS)

then in the model:

    live = LiveEntryManager()
    objects = models.Manager()

When a model has a custom manager, Django doesn’t automatically set up the objects manager for you.

The first manager defined in a model class is given special status. It becomes the default manager for that model, in addition to the name it was defined with. It will also be available as the attribute `_default_` manager, so you can actually write this as:

    def render(self, context):
        context[self.varname] = self.model._default_manager.all()[:self.num]
        return ''

### extending the default manager
Create a manager.py file

    from django.db import models
    from django.contrib.auth.models import User
    from django.db.models import Count

    class SnippetManager(models.Manager):
        def top_authors(self):
        return User.objects.annotate(score=Count('snippet')).order_by('score')

Then in the model

    from myapp import managers

    ...

    class MyModel(models.Model):
        ...
        objects = managers.SnippetManager()

Now inside a view

    def top_authors(request):
        return object_list(request, queryset=Snippet.objects.top_authors(),
            template_name='cab/top_authors.html',
            paginate_by=20)

## Querystring

### Delete objects

        Bookmark.objects.filter(user__pk=request.user.id, snippet__pk=snippet.id).delete()

Instead of querying to see if the user has a bookmark for this snippet and then deleting it manually (which incurs the overhead of two database queries), you simply use filter() to create a QuerySet of any bookmarks that match this user and this snippet. You then call the delete() method of that QuerySet. This issues only one query—a DELETE query, whose FROM clause limits it to the correct rows, if any exist.

### aggregate
Previously, you used the annotate method, because you needed to add an extra piece of information to the results returned by the query. But now you just want to directly return the aggregated value and nothing else, so you’ll use a different method called aggregate. If you have a Snippet object in a variable named snippet, and you want the sum of all the ratings attached to it, you can write the query like this: (the `Rating` model has a `Snippet` `ForeignKey` field)

    from django.db.models import Sum
        total_rating = snippet.rating_set.aggregate(Sum('rating'))

## Forms

### required/optional
All of the field types built into Django’s form system are required by default and so cannot be left blank. If either of the password fields were left blank, Django would raise a ValidationError before calling the clean() method, so you wouldn’t need to raise an additional error to require a value. To mark a form field as optional, pass it the keyword argument required=False.

### validation
The form `is_valid()` method performs the following stack

- `full_clean()`
- Field `clean()`
- Form `clean_FIELDNAME()`
- Form `clean()`
- errors or cleaned data

1. `full_clean()` loops through the fields on the form. Each field class has a method named `clean()`, which implements that field’s built-in validation rules, and each of these methods will either raise a `ValidationError` or return a value. If a ValidationError is raised, no further validation is done for that field. If a value is returned, it goes into the form’s `cleaned_data` dictionary.
2. If a field’s built-in clean() method didn’t raise a ValidationError, then any available custom validation method is called. Again, these methods can either raise a ValidationError or return a value; if they return a value, it goes into `cleaned_data`.
3. Finally, the form’s `clean()` method is called. It can also raise a ValidationError, also one that’s not associated with any specific field. If clean() finds no new errors, it should return a complete dictionary of data for the form, usually by doing return `self.cleaned_data`.
4. If no validation errors were raised, the form’s `cleaned_data` dictionary will be fully populated with the valid data. If there were validation errors, however, `cleaned_data` will not exist, and a dictionary of errors (`self.errors`) will be filled with validation errors. Each field knows how to retrieve its own errors from this dictionary, which is why you can do things like `{{ form.username.errors }}` in a template.
5. Finally, `is_valid()` returns either False if there were validation errors or True if there weren’t.

## Templates

### how they work
Before you can dive into writing your own custom extensions to Django’s template system, you
need to understand the actual mechanism behind it. Knowing how things work “under the
hood” makes the process of writing custom template functionality much simpler.
The process Django goes through when loading a template works—roughly—like this:
- Read the actual template contents: Most often this means reading out of a template file on disk, but that’s not always the case. Django can work with anything that hands over a string containing the contents you want it to treat as a template.
- Parse through the template, looking for tags and variables: Each tag in the template, including all of Django’s built-in tags, will correspond to a particular Python function defined somewhere (inside django/template/defaulttags.py in the case of the built-in tags). You’ll see in a moment how to tell Django that a particular tag maps to a par- ticular function. Typically this function is referred to as the tag’s compilation function because it’s called while Django is compiling a list of the eventual template contents.
- For each tag, call the appropriate function, passing in two arguments: One argument is the parsing class that is reading the template (useful for doing tricky things with the way the template gets processed), and the other is a string containing the contents of the tag. So, for example, the tag {% if foo %} results in Django passing a function (called do_if(), in Django’s default tag library) an instance of the parsing class and an object that holds the tag contents “if foo.”
- Make a note of the return value of the Python function called for each tag: Each func- tion is required to return an instance of a special class—django.template.Node—or a subclass of it, and choosing an appropriate Node subclass based on the particular tag. 

The result is an instance of the class django.template.Template, which contains a list of
Node instances (or instances of Node subclasses). This is the actual “thing” that will be rendered
to produce the output. Each Node is required to have a method named render(), which accepts
a copy of the current template context (the dictionary of variables available to the template) and
returns a string. The output of the template comes from concatenating those strings together.

### inheritance
within a block, you’ll have access to the content that
would have gone there if you weren’t supplying your own. This content is stored in a special
variable named `block.super`. So if you had a base template that contained this:

    {% block title %}My weblog:{% endblock %}

you could write a template that extended it, and fill in your own content:

    {% block title %}My page{% endblock %}

Using block.super, you could access the default content from the parent block to get a
final value of My weblog: My page:

    {% block title %}{{ block.super }} My page{% endblock %}

### 3 layered structure
- single base template containing the common HTML of all pages.
- Section-specific base templates that fill in appropriate navigation and/or theming. These extend the base template.
- The “actual” templates that will be loaded and rendered by the views. These extend the appropriate section-specific templates.

### tips

    <body class="{% block bodyclass %}{% endblock %}">

A common technique in CSS-based web design is to use a class attribute on the body tag
to trigger changes to a page’s style. For example, you’ll have a list of navigation options in the
sidebar, representing different parts of the blog—entries, links, and so forth—and it would be
nice to highlight the part a visitor is currently looking at. By changing the class of the body tag
in different parts of the site, you can easily use CSS to highlight the correct item in the navigation list.

### tags
Due to the way Django’s template inheritance works, a custom tag or filter library loaded via the `{% load %}` tag will be available only in the block in which it was loaded. If you need to reuse the same tag library in a different block, you’ll need to load it again.

#### parse

If we want to create a templatetag to be used this way:

    {% if_bookmarked user object %}
        <form method="post" action="{% url cab_bookmark_delete object.id %}">
        <p><input type="submit" value="Delete bookmark"></p>
        </form>
    {% else %}
        <p><a href="{% url cab_bookmark_add object.id %}">Add bookmark</a></p>
    {% endif_bookmarked %}

we need to use the parser object passed to the compilation function

    def do_if_bookmarked(parser, token):
        bits = token.contents.split()
        if len(bits) != 3:
            raise template.TemplateSyntaxError("%s tag takes two arguments" % bits[0])

So we search for all nodes that stays inside the True part (before the `else` tag or the `endif_bookmarked` tag)

    nodelist_true = parser.parse(('else', 'endif_bookmarked'))

The call to parser.parse() moves ahead in the template to just before the first item in the list you told it to look for. This means you now want to look at the next token and find out if it’s an {% else %}. If it is, you’ll need to do a bit more parsing

    token = parser.next_token()
        if token.contents == 'else':
            nodelist_false = parser.parse(('endif_bookmarked',))
            parser.delete_first_token()
        else:
            nodelist_false = template.NodeList()

Finally, you return a Node class, passing the two arguments gathered from the tag and the two NodeList instances

    return IfBookmarkedNode(bits[1], bits[2], nodelist_true, nodelist_false)

Now the node class:

    class IfBookmarkedNode(template.Node):
        def __init__(self, user, snippet, nodelist_true, nodelist_false):
        self.nodelist_true = nodelist_true
        self.nodelist_false = nodelist_false

right now the `user` and `snippet` variables are string!
You need some way of saying that these are actually template variables that you need to resolve later. Fortunately, that’s easy enough to do: 

    self.user = template.Variable(user) self.snippet = template.Variable(snippet)

The Variable class in django.template handles the hard work for you. When given the template context to work with, it knows how to resolve the variable and gives you back the actual value it corresponds to. Now you can start to write the render() method:

    def render(self, context):
        user = self.user.resolve(context)
        snippet = self.snippet.resolve(context)

Each Variable instance has a method called `resolve()`, which handles the actual business of resolving the variable. If the variable turns out not to correspond to anything, it’ll even handle raising an exception `django.template.VariableDoesNotExist` automatically for you. Of course, you’ve seen that it’s usually a good idea for custom template tags to fail silently.
You have two NodeList instances, and you want to render one or the other according to whether the user has bookmarked the snippet. Fortunately, that’s easy. Just as a Node must have a `render()` method that accepts the context and returns a string, so too must NodeList:

    if Bookmark.objects.filter(user__pk=user.id, snippet__pk=snippet.id):
        return self.nodelist_true.render(context)
    else:
        return self.nodelist_false.render(context)

To register the tag:

    register = template.Library()
    register.tag('if_bookmarked', do_if_bookmarked)

### filters

#### pluralize

    <p>This entry is part of the
    categor{{ object.categories.count|pluralize:"y,ies" }}

#### timesince

    <li>
        <a href="{{ entry.get_absolute_url }}">{{ entry.title }}</a>,
        posted {{ entry.pub_date|timesince }} ago.
    </li>

## Context Processors

RequestContext gets its name from the fact that it makes use of functions called context processors. Each context processor is a function that receives a Django `HttpRequest` object as an argument and returns a dictionary of variables based on that `HttpRequest`. `RequestContext` then automatically adds those variables to the context, in addition to any variables explicitly passed to the context during the process of executing a view function.
In normal use, RequestContext reads its list of context-processor functions from the setting TEMPLATE_CONTEXT_PROCESSORS. The default set happens to include a context processor that reads `request.user` to get the current user and adds it to the context as the variable `{{ user }}`.
Django’s generic views by default uses `RequestContext`.

## Contrib

### markup

To enable the filter, you’ll need to add one more entry to your `INSTALLED_APPS` setting:
`django.contrib.markup`, which contains tools for working with common text-to-HTML translation systems (including Markdown).

    {% load markup %}
    <blockquote>{{ comment|markdown: "safe" }}</blockquote>

This will apply Markdown to the comment’s contents, and will also enable Markdown’s “safe mode,” which strips any raw HTML tags out of the comment before generating the final HTML to display.

## E-mail
    MANAGERS = (('Alice Jones', 'alice@example.com'),
        ('Bob Smith', 'bob@example.com'))

In other words, it’s a tuple, or list of tuples, where each tuple contains a name and an
e-mail address. When these are filled in, two functions in `django.core.mail—mail_admins()`
and `mail_managers()` can be used as a shortcut to send an e-mail to those people.

## Develping multiple applications

### Split off features in different applications

When is it better to separate features in different applications?

Think in terms of orthogonality. Generally, in software development, two features are orthogonal if a change to one doesn’t affect the other. User preferences, then, are orthogonal to user signups, because you could, for example, change the way the signup process works (say, by adding an explicit activation step or building in mea- sures to defeat spambots) without changing the way users configure their preferences. When features are clearly orthogonal to each other like this, they almost always belong in separate applications.

Finally, reuse can be a good criterion for determining whether some particular feature deserves to be split out into its own application. If you can imagine a case where you’d want to use that feature, and just that feature, on another site, the odds are good that it ought to be in a separate application to make that reuse easier.

### Flexible form handling

The view

    def contact_form(request, form_class=ContactForm):
        if request.method == 'POST':
            form = form_class(data=request.POST)
            if form.is_valid():
                form.save()
                return HttpResponseRedirect("/contact/sent/")
            else:
                form = form_class()
            return render_to_response('contact_form.html',
                { 'form': form },
                context_instance=RequestContext(request))

The URL pattern

    url(r'^inquiries/sales/$',
        contact_form,
        { 'form_class': SalesInquiryForm },
        name='sales_inquiry_form'),

### Flexible template handling

View

    def contact_form(request, form_class=ContactForm, template_name='contact_form.html'):
        if request.method == 'POST':
            form = form_class(data=request.POST)
            if form.is_valid():
                form.save()
                return HttpResponseRedirect("/contact/sent/")
            else:
                form = form_class()
            return render_to_response(template_name,
                { 'form': form },
                context_instance=RequestContext(request))

The URL pattern

    url(r'^inquiries/sales/$',
        contact_form,
        { 'form_class': SalesInquiryForm,
        'template_name': 'sales_inquiry.html' },
        name='sales_inquiry_form'),

### Flexible Post-Form Processing

Define a `success_url`

    def contact_form(request, form_class=ContactForm,
        template_name='contact_form.html',
        success_url='/contact/sent/'):
        ...

### More

I generally try to support at least the following arguments:

- `form_class`, when I’m writing a view that handles a form
- `success_url`, when I’m writing a view that redirects after successful processing (of a form, for example)
- `template_name`, as in generic views
- `extra_context`, also as in generic views

Also, I always make sure to use RequestContext for template rendering. This enables both the standard set of context processors, which add things to the context like the identity of the currently logged-in user, as well as any custom context processors that have been added to the site’s settings.

## Distributing django applications

Sometimes you will have code that’s tightly coupled to a particular project. For example, it’s somewhat common to write a view that handles the home page of a site, and have that view handle requirements that are so site-specific that it wouldn’t make sense to reuse that view in other projects. If you’d like, you can place code like this in an application that’s directly inside the project directory, but keep in mind that for common cases like this, there’s no need for an application. Django doesn’t require that view functions be within an application module (Django’s own generic views aren’t, for example). So you can simply put project-specific views directly inside the project. You only need to create an application if you’re also defining models or custom template tags.

### Python Packaging tools

Because a Django application is just a collection of Python code, you should simply use standard Python packaging tools to distribute it. The Python standard library includes the module `distutils`, which provides the basic functionality you’ll need: creating packages, installing them, and registering them with the Python Package Index (if you want to distribute your application to the public).

The primary way you’ll use distutils is by writing a script—conventionally called setup.py—that contains some information about your package. Then you’ll use that script to generate the package. In the simplest case, this is a three-step process:

- In a temporary directory (not one on your Python import path), create an empty setup.py file and a copy of your application’s directory, containing its code.
- Fill out the setup.py script with the appropriate information.
- Run python setup.py sdist to generate the package; this creates a directory called dist that contains the package.

The other common method of distributing Python packages uses a system called setuptools. setuptools has some similarities to distutils But setuptools adds a large number of features on top of the standard distutils, including ways to specify dependencies between packages and ways to automatically download and install packages and all their dependencies. You can learn more about setuptools online at http://peak.telecommunity.com/DevCenter/setuptools

#### setup.py

    from distutils.core import setup
    setup(name="hello",
        version="0.1",
        description="A simple packaged Python application",
        author="Your name here",
        author_email="Your e-mail address here",
        url="Your website URL here",
        py_modules=["hello"],
        download_url="URL to download this package here")

Now you can run python setup.py sdist, which creates a dist directory containing a file named hello-0.1.tar.gz

The installation process is simple: open up the package (the file is a standard compressed archive file that most operating systems can unpack), and it will create a directory called hello-0.1 containing a setup.py script. Running python setup.py install in that directory installs the package on the Python import path.

To handle multiple modules or submodules, you simply list them in the py_modules argument. For example, if you have an application named foo, which contains a submodule named foo.templatetags, you’d use this argument to tell distutils to include them:

    py_modules=["foo", "foo.templatetags"],

Including LICENSE, README, etc.. files in a Python package is fairly easy. While the setup.py script specifies the Python modules to be packaged, you can list additional files like these in a file named MANIFEST.in (in the same directory as setup.py). The format of this file is extremely simple and often looks something like this:

    include LICENSE.txt
    include README.txt
    include CHANGELOG.txt

Each include statement goes on a separate line and names a file to be included in the package. For advanced use, such as packaging a directory of documentation files, you can use a recursive-include statement. For example, if documentation files reside in a directory called docs, you could use this statement to include them in the package:

    recursive-include docs *

## Documentation

### Documentation Displayed Within Django

This last point is particularly important, because Django can sift through your code for docstrings and use them to display useful documentation to users. The administrative interface usually contains a link labeled “Documentation” (in the upper right-hand corner of the main page), which takes the user to a page listing all of the documentation Django can produce (if the necessary Python documentation tools are available; see the next section for details).

- A list of all the installed models, organized by the applications they belong to: Foreach model, Django shows a table listing the fields defined on the model and any custom methods, as well as the docstring of the model class.
- A list of all the URL patterns and the views they map to: For each view, Django displays the docstring.
- Lists of all available template tags and filters, both from Django’s own built-in set and from any custom tag libraries include

### What to document

For example, you don’t need to supply a docstring for the `get_absolute_url()` method of a model, because that’s a standard method to define on models, and you can trust that people reading your code will know why it’s there and what it’s doing. However, if you’re providing a custom `save()` method, you often should document it, because an explanation of any special behavior it provides will be useful to people reading your code.

### Good practice

- Model classes should include information about any custom managers attached to the model: However, they don’t need to include a list of fields in their docstrings, because that’s generated automatically.
- Docstrings for view functions should always mention the template name that will be used: In addition, they should provide a list of variables that are made available to the template.
- Docstrings for custom template tags should explain the syntax and arguments the tags expect: Ideally, they should also give at least one example of how the tag works.

If you’d like, you can use REstructured text syntax in your docstrings to get nicely formatted documentation.

### Django specific

    def latest_entries(request):
        """
        View of the latest 15 entries published. This is similar to
        the :view:'django.views.generic.date_based.archive_index'
        generic view.
        **Template:**'
        ''coltrane/entry_archive.html''
        **Context:**
        ''latest''
        A list of :model'coltrane.Entry' objects.
        """
        return render_to_response('coltrane/entry_archive.html',
            { 'latest': Entry.live.all()[:15] })

In addition to the :view: and :model: shortcuts shown in the previous example, three others are available:

- :tag:: This should be followed by the name of a template tag. It links to the tag’s documentation.
- :filter:: This should be followed by the name of a template filter. It links to the filter’s documentation.
- :template:: This should be followed by a template name. It links to a page that either shows locations in your project’s `TEMPLATE_DIRS` setting where that template can be found, or shows nothing if the template can’t be found.
