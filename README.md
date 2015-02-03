# Practical Django Projects

Just some extracts from the "Practical Django Projects" book which capture my attention

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
