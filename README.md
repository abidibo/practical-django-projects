# Practical Django Projects

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
