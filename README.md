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

Read http://www.rrn.dk/the-difference-between-utf-8-and-unicode

## Models

### blank and null
Also, it’s important to note that for text-based field types (CharField, TextField, and others), Django
will never insert a NULL. For these field types, a blank value will be inserted as an empty string. This is to
avoid a situation where there are potentially two different blank values for the field (either an empty string or
a NULL) and to ensure that code that checks for blank values can be kept simple. Because of this, you should
generally avoid specifying null=True on text-based field types.

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
