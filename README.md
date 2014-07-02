WARNING: This readme is out of date in the development branch, please refer to [carteblanche-django-starter](https://www.google.com) for an example project that demonstrates 

python-carteblanche
===================

A menuing system of unlimited power. Holds and serializes the in-memory relationship between urls and objects and users.


Installation
------------
You can obtain the source code for carteblanche from here:

    https://github.com/neuman/python-carteblanche

Or install it with pip from the console

	pip install carteblanche

Usage
-----

###Use in a model

```python
from django.db import models
from django.core.urlresolvers import reverse
from carteblanche.models import Verb, Noun

class ProjectVerb(Verb):
    display_name = "Human Readable Verb Name"
    def is_available(self, user):
        #insert your own conditional logic here to determine 
        #if this user has permission to do this action
        if self.noun.owners.filter(id=user.id).count() > 0:
            return True
        else:
            return False

    def get_url(self):
        #I use django reverse to spit out the url of the action 
        #for the given model but you can substitute your own logic
        return reverse(
            viewname='project_action_view_name', 
            args=[self.noun.id], 
            current_app='app_name'
            )

class Project(models.Model, Noun):
    owners = models.ManyToManyField(Person)
    verb_classes = [ProjectVerb]

```

###Use in a view 

```python
    class ProjectsView(TemplateView, Noun):
        template_name = 'whatever.html'
        verb_classes = [ProjectCreateAction]

        def get_context_data(self, **kwargs):
            context = super(ProjectsView, self).get_context_data(**kwargs)
            context['available_actions'] = self.get_available_verbs(self.request.user)
            return context
```

###Use inheretance to avoid running the same query twice

If some of your verbs need to run the same query to check availability, you can specify an `avalability_key` and `get_available_verbs` will store the value returned from that verb's `is_available` method.  Any verbs that are checked after that with the same `availability_key` will use the stored value.  Inheretance is a simple way to do this without rewriting the `avalability_key` or shared `is_available` method.

```python
class ProjectMemberVerb(Verb):
    availability_key = "is_member"
    def is_available(self, user):
        return self.noun.is_member(user)

class ProjectUploadVerb(ProjectMemberVerb):
    display_name = "Upload Media"

    def get_url(self):
        return "projects/upload"

class ProjectPostVerb(ProjectMemberVerb):
    display_name = "Post"

    def get_url(self):
        return "/projects/post"

class Project(Noun):
    run_count = 0
    verb_classes = [ProjectUploadVerb, ProjectPostVerb]

    def is_member(self, user):
        self.run_count += 1
        return True
```

###You can override the 'get_verbs'  method if you have custom logic.

```python
class Project(models.Model, Noun):
    owners = models.ManyToManyField(Person)

    def get_verbs(self):
        verbs = [
            ProjectVerbs(self)
        ]
        return verbs
```

###Displaying in a Template

```html
<ul>
  {% for verb in noun.get_available_verbs %}
      <li><a href="{{ verb.url }}">{{ verb.display_name }}</a></li>
  {% endfor %}
</ul>
```

