Flask: Write to a Database
==========================

In this chapter we focus on writing data to a SQLite database, using Flask-SQLAlchemy.

We will cover:

1. Add a new record: Create a complete new entry and add it to the database.
2. Update a record: Retrieve an existing record and allow the user to edit any part of it, then write the changes to the database.
3. Delete a selected record.

Resources:

* `SQLAlchemy documentation <https://www.sqlalchemy.org/>`_
* `Flask-SQLAlchemy documentation <https://flask-sqlalchemy.palletsprojects.com/>`_
* `Code for this chapter <https://github.com/macloo/python-adv-web-apps/tree/master/python_code_examples/flask/databases/flask_db_write>`_

The database
------------

We will use the same SQLite database from the chapter `Flask: Read from a Database <flask_db2.html>`_.

The database is named *sockmarket.db.* It has only one table, named **socks.** It has seven fields: *id, name, style, color, quantity, price,* and *updated.* You will see the table name and the field names later, in the Python code. The image below is a screenshot from the `DB Browser for SQLite <https://sqlitebrowser.org/>`_, showing the top rows of the **socks** table.

.. figure:: _static/images/socks_db_browser.png
   :alt: Socks table in DB Browser screenshot

It is essential to get your **database connection** working without errors before you try to do more with the database and Flask.

* Refer to `Flask and Databases <flask_db1.html>`_ to test your database connection.

Add new record to database
--------------------------

To add a new record to the database as part of a Flask app, you will likely use an HTML form similar to this:

.. figure:: _static/images/sock_entry_form_ss.png
   :alt: Add a new record form screenshot

The route: *localhost:5000/add_record*

Flask forms using **Flask-WTF** and **Flask-Bootstrap4** were introduced in the chapter `Flask: Web Forms <flask_forms.html>`_. See `the code for this form <#the-form-without-validators>`_ below.

App setup
+++++++++

The example app we will build here begins with the app completed in the chapter about `reading from a database <flask_db2.html>`_. Now there will be a lot of *new imports* at the top of the app file:

.. literalinclude:: ../python_code_examples/flask/databases/flask_db_write/write_db.py
   :linenos:
   :lines: 1-29
   :caption:

* Line 3: new imports from Flask — ``request`` and ``flash``. You’ll see these in the first new route.
* Line 8: new form fields you have not seen before, because the form here has a lot more going on than the simple form in the introductory chapter.
* Line 9: new form validators to prevent unwanted values going into your database.
* Line 10: import to enable Python to insert a date automatically into your database.
* Line 15: required by WTForms; covered in the forms chapter.
* Everything else above was covered in previous chapters.

Database table setup
++++++++++++++++++++

.. literalinclude:: ../python_code_examples/flask/databases/flask_db_write/write_db.py
   :linenos:
   :lines: 30-50
   :emphasize-lines: 4,14
   :lineno-start: 30
   :caption:

Lines 33–41 were explained in the chapter about reading from a database. Lines 43–49 were not needed there, but now we need them, or we’ll get an error when we try to write a new record into this table in the database.

``__init__()`` is a reserved method in Python classes. It is called when an object is created from the class, to initialize the properties of the class. Here, it specifies that when we pass values into **a new record** in this table, **the order of the values** will be *name, style, color, quantity, price, updated.*  **Note** that the order matters! You’ll see this in the first new route. ``self`` represents the new instance of the class, and everywhere you see it, it needs to stay.

.. important:: If your app needs to write a new record into *more than one* table, you’ll need to set up each of those tables in this way also.

The form without validators
+++++++++++++++++++++++++++

As explained in the web forms chapter, we can vastly simplify **the template HTML** by defining every aspect of a form in a new class **in the app script.**

.. code-block:: python

    class AddRecord(FlaskForm):
        # id used only by update_record
        id_field = HiddenField()
        name = StringField('Sock name')
        style = SelectField('Choose the sock style',
            choices=[ ('', ''), ('ankle', 'Ankle'),
            ('knee-high', 'Knee-high'),
            ('mini', 'Mini'),
            ('other', 'Other') ])
        color = StringField('Color')
        quantity = IntegerField('Quantity in stock')
        price = FloatField('Retail price per pair')
        # updated - date - handled in the route function
        updated = HiddenField()
        submit = SubmitField('Add/Update Record')

.. note:: Each different field type here was *imported* from **wtforms** at the top of the script file.

`See a list of all WTForms field types. <https://github.com/macloo/python-adv-web-apps/blob/master/python_code_examples/flask/forms/WTForms-field-types.csv>`_ Note that they are *case-sensitive.*

The following might be puzzling:

* ``choices=[]`` — required for a SelectField. Inside the square brackets are tuples. The first item in a tuple is the value for the database. The second item in a tuple is the text visible in the select menu.
*  ``HiddenField()`` denotes an HTML form element: ``<input type="hidden">``. `Learn more at MDN. <https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/hidden>`_

Everything else should be clear if you are familiar with regular HTML forms. Compare the code to `the screenshot of the form <#add-new-record-to-database>`_.

The form with validators
++++++++++++++++++++++++

Validators — also imported from **wtforms** — can get pretty complicated. They can do quite a lot to prevent bad values from being written to your database, though, so it’s worthwhile to learn how to use them.

Below is *the same form class* with the validators added. This is the class used in the example app.

`See a list of all WTForms validators. <https://github.com/macloo/python-adv-web-apps/blob/master/python_code_examples/flask/forms/WTForms-validators.csv>`_

.. code-block:: python

    class AddRecord(FlaskForm):
        # id used only by update_record
        id_field = HiddenField()
        name = StringField('Sock name', [ InputRequired(),
            Regexp(r'^[A-Za-z\s\-\']+$', message="Invalid sock name"),
            Length(min=3, max=25, message="Invalid sock name length")
            ])
        style = SelectField('Choose the sock style', [ InputRequired()],
            choices=[ ('', ''), ('ankle', 'Ankle'),
            ('knee-high', 'Knee-high'),
            ('mini', 'Mini'),
            ('other', 'Other') ])
        color = StringField('Color', [ InputRequired(),
            Regexp(r'^[A-Za-z\s\-\'\/]+$', message="Invalid color"),
            Length(min=3, max=25, message="Invalid color length")
            ])
        quantity = IntegerField('Quantity in stock', [ InputRequired(),
            NumberRange(min=1, max=999, message="Invalid range")
            ])
        price = FloatField('Retail price per pair', [ InputRequired(),
            NumberRange(min=1.00, max=99.99, message="Invalid range")
            ])
        # updated - date - handled in the route
        updated = HiddenField()
        submit = SubmitField('Add/Update Record')

* ``InputRequired()`` means the field cannot be left empty.
* ``Regexp()`` is used to provide a *regular expression,* which is a *pattern* the entered string needs to match. `Learn about regular expressions in Python. <https://docs.python.org/3/howto/regex.html>`_ The ``message=`` value inside the parentheses will appear if the pattern is not matched.
* ``Length()`` allows us to provide a minimum and maximum length for the string. It also provides a ``message=`` option.
* ``NumberRange()`` allows us to provide a minimum and maximum number for an integer or a float.

The route function
++++++++++++++++++

The route — like all the code shown above — is in the app script, *write_db.py.*

The route *add_record* renders a template that contains the web form discussed above. In the decorator we add ``methods=['GET', 'POST']`` because this template can legitimately open via a regular HTTP request (*get*) or via a request with form data attached to it (*post*).

The function has an if/else to handle two different circumstances:

1. The form on the page has been submitted, and the form data were all valid. (The ``validate_on_submit()`` function returns True.) In that case, we want to send the data to the database.
2. No data will be written to the database. That case actually has two possible conditions:

    a. The form has not yet been filled in and submitted.
    b. The form was submitted, but there is invalid data in one or more fields.

First, we’ll look at what happens if the data are valid and are written to the database:

.. code-block:: python
    :linenos:
    :emphasize-lines: 4,19

    # add a new sock to the database
    @app.route('/add_record', methods=['GET', 'POST'])
        def add_record():
            form1 = AddRecord()
            if form1.validate_on_submit():
                name = request.form['name']
                style = request.form['style']
                color = request.form['color']
                quantity = request.form['quantity']
                price = request.form['price']
                # get today's date from function, above all the routes
                updated = stringdate()
                # the data to be inserted into Sock model - the table, socks
                record = Sock(name, style, color, quantity, price, updated)
                # Flask-SQLAlchemy magic adds record to database
                db.session.add(record)
                db.session.commit()
                # create a message to send to the template
                message = f"The data for sock {name} has been submitted."
                return render_template('add_record.html', message=message)
            else:

Line numbers start at 1 here, but not in the app.

* Line 4: Using `the form class created above <#the-form-without-validators>`_, we create an *instance* of that form and assign it to the variable ``form1``.
* Line 5: The ``if`` clause runs if the form data were valid. That means we want to write the data to the database.
* Lines 6–10: Create new variables from the data received from the form.
* Line 12: Run a function to create a date string representing today’s date. Assign that string to the new variable ``updated``.
* Line 14: Using the `model class created above <#database-table-setup>`_ — ``Sock`` — we format the data in the manner expected by the database, following the order in the ``__init__`` function. Assign this to a new variable, ``record``.
* Lines 16–17: This is SQLAlchemy handling everything to securely write to the database. Note that the variable ``record`` is used in line 16. ``db.session.commit()`` is necessary to update your database.
* Line 19: Create a message to pass to the template. Note that this message *only exists if* the data were sent to the database.
* Line 20: Render the template, passing the message to it.

.. figure:: _static/images/submit_success.png
   :alt: Result of successful submit in browser screenshot

Above, the result of successfully writing to the database renders the template *without showing the form.* It displays the message, including the name of the sock just submitted.

**What happens if data in one or more form fields are invalid when the form is submitted?**

Recall that in `the form class <#the-form-with-validators>`_, *validators* were added to restrict what can be sent to the database.

.. figure:: _static/images/errors_in_form.png
   :alt: Result of unsuccessful submit in browser screenshot

Above, you can see the result of **invalid data** in two of the form fields. The user can correct the errors and submit the form again. This is handled in the second part of the route function — the *else* clause:

.. code-block:: python
    :linenos:
    :lineno-start: 21
    :emphasize-lines: 9

    else:
        # show validaton errors
        for field, errors in form1.errors.items():
            for error in errors:
                flash("Error in {}: {}".format(
                    getattr(form1, field).label.text,
                    error
                ), 'error')
        return render_template('add_record.html', form1=form1)

See the `Flask Flash Function Tutorial <https://pythonprogramming.net/flash-flask-tutorial/>`_ for details about using ``flash()`` and the for-loop in the code above.

Note that the *else* clause also runs if this page is freshly opened in the browser, with an empty form. The for-loop runs, but there are no errors, and so no ``flash()`` messages will appear on the page.

The form — ``form1`` — *must* be passed to the template in the return statement (line 29 above).

The template
++++++++++++

Because of the ``flash()`` messages, this template is a bit complicated.



.