# Mongoose Forms

A form templating and validation library for Mongoose ODM using Handlebars templates and node validator

## Installing

    $ npm install mongoose-forms

## Usage:

### Create a Form from a Model - Simple

```javascript
var Form    = require('mongoose-forms').Form;
var Model   = require('../models/Model.js')

var form = mongooseForms.Form(Model); // Form fields will be generated from schema
                                      // with default values and type detection

```

### Render Using Handlebars

Register our helpers

```javascript
mongooseForms.bindHelpers(Handlebars, 'bootstrap'); // Using the bootstrap markup style
```

Call from inside template

```html
<div class="container">
    {{{renderForm formObject}}}
</div>
```

### Save Model from Form (showing Express POST route) 

```javascript
var Bridge = require('mongoose-forms').Bridge;

app.post('/site/create', function(req, res) {
  
  var form = SiteForm();

  if(!form.isValid(req.body)) { ... Rerender form, automatically showing errors ... }

  Bridge(new Site, form) // Bridge populates Model from Form
    .getModel()
    .save(function(err, site) { });
});
```

### Populate Form from Model (showing Express GET route)

```javascript
app.get('/site/:id', function(req, res) {

  Site.findById(req.params.id, function(err, site) {

    var form = Bridge(site, new SiteForm).getForm(); // bridge populates Form from Model
    
    res.render('update', { form: form });
  });
});
```

### Create a Form from a Model - Advanced

```javascript
var forms   = require('mongoose-forms');
var Site    = require('../models/Site.js')

var form = forms.Form(Site, {
  renderOuter: true,          // render the form container
  class: 'form-horizontal',   // give the form a class
  legend: 'Site Name',        // render a legend (only if renderOuter: true)
  maps: ['name', 'slug'],     // map only to these members of model
  method: 'post',             // form method
  fields: {
    actions: {                // define custom fields that may or may not exist in your model
      template: 'Actions',    // provide a template name
      order: 100,             // order acts like a weight
      buttons: [              // custom fields can take arbitrary data
        {
          type: 'submit',
          label: 'Submit Form',
          'class': 'btn-primary'
        }
      ]
    },
    name: { // We can define validation and return sanitized data (or return nothing to simply passthrough)
      validate: function(value, check, sanitize) {
        check(value, 'Must be at least 6 characters').len(6);
        check(value, "Can't be more than 15 characters").len(6, 15);
        return sanitize(value).xss().trim();
      }
    }
  }
});

```

### Simplify life with a builder (name it something like: forms/User.js)

var forms = require('mongoose-forms');
var User  = require('../models/User.js');

```javascript
exports.Form = function() {
  return forms.Form(User, {
    method: 'post',
    action: '/user/create',
    maps: ['username', 'password'],
    fields: {
      password: {
        template: 'Password',
        validate: function(value, check) {
          check(value, 'Minimum 6 characters and maximum 10').len(6, 10);
        }
      },
      submit: {
        template: 'Submit'
      }
    }
  });
}
```

```javascript
var UserForm = require('./lib/forms/User.js').Form;
```