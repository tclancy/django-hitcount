Django-HitCount
===============

Updated to work with Django 1.6 and Python 3.

Basic app that allows you to track the number of hits/views for a particular
object.

For original docs see:

<http://blog.damontimm.com/django-hitcount-app-count-hits-views/>

What it is not
--------------

This is not meant to be a user tracking app (see: [django-tracking][1]) or a
comprehensive site traffic monitoring tool (see: Google Analytics).

It's meant to serve as a simple hit counter for chosen objects with a couple
useful features (user-agent, session, and IP tracking) and tools to help you
on your way.

Installation:
-------------

Simplest way to formally install is to run:

    ./setup.py install

Or, you could do a PIP installation:

    pip install -e git://github.com/thornomad/django-hitcount.git#egg=django-hitcount

Or, you can link the source to your `site-packages` directory.  This is useful
if you plan on pulling future changes and don't want to keep running
`./setup.py install`.

    git clone git://github.com/thornomad/django-hitcount.git
    ln -s `pwd`/django-hitcount/hitcount `python -c "from distutils.sysconfig import get_python_lib; print(get_python_lib())"`/hitcount

Special thanks to ariddell for putting the `setup.py` package together.

[1]:http://code.google.com/p/django-tracking/

Use
---

Taken from above link, reproduced and updated as necessary here:

First test the installation by starting a django shell and seeing the following:
    >>> import hitcount
    >>> hitcount.__version__
    '0.2 beta'

### Adding to your Django Project

Add the hitcount app to your INSTALLED_APPS tuple and run syncdb.

There are three additional settings you can add to your settings.py file:

    HITCOUNT_KEEP_HIT_ACTIVE = { 'days': 7 }
    HITCOUNT_HITS_PER_IP_LIMIT = 0
    HITCOUNT_EXCLUDE_USER_GROUP = ( 'Editor', )

HITCOUNT_KEEP_HIT_ACTIVE: is the number of days, weeks, months, hours, etc (timedelta kwargs), that an Hit is kept ‘active’. If a Hit is ‘active’ a repeat viewing will not be counted. After the active period ends, however, a new Hit will be recorded. You can decide how long you want this period to last …

HITCOUNT_HITS_PER_IP_LIMIT: limit the number of ‘active’ hits from a single IP address. 0 means that it is unlimited. You may want to set this, or not.

HITCOUNT_EXCLUDE_USER_GROUP: don’t count any hits from certain logged in users. In the example above, I don’t want any of my editors inflating the total Hit count.

### Adding to your urls.py

You need to add one line to a suitable urls.py file to support the url used to call back from the client browser with the hit.

You can have this url point to anywhere you like, but you need to keep the name='hitcount_update_ajax' constant.

from django.conf.urls.defaults import *
from django.views.generic.list_detail import object_detail
from hitcount.views import update_hit_count_ajax

urlpatterns = patterns('',
    url(r'^hitcount_update/$', # you can change this url if you would like
        update_hit_count_ajax,
        name='hitcount_update_ajax'), # keep this name the same

        ...

### Set hit counting in your template

Add the javascript to your object_detail templates (or any template that handles a single object) so that our hit counter is called after the document loads.

Here is what my head includes:

    {% load hitcount_tags %}
    <script src="/media/js/jquery-latest.js" type="text/javascript"></script>
    <script type="text/javascript"><!--
        $(document).ready(function() {
            {% get_hit_count_javascript for object %}
        });
    --></script>
When the template is rendered, it should turn into something like this:

    $(document).ready(function() {

    $.post( '/hitcount_update/',
        { hitcount_pk : '3' },
        function(data, status) {
            if (data.status == 'error') {
                // do something for error?
            }
        },
        'json');

    });

### Display the hits on the front end!
The most exciting part, is actually displaying your hits. There are four different ways to do it:

    - Return total hits for an object:
      {% get_hit_count for [object] %}

    - Get total hits for an object as a specified variable:
      {% get_hit_count for [object] as [var] %}

    - Get total hits for an object over a certain time period:
      {% get_hit_count for [object] within ["days=1,minutes=30"] %}

    - Get total hits for an object over a certain time period as a variable:
      {% get_hit_count for [object] within ["days=1,minutes=30"] as [var] %}

### Display the hits on admin!