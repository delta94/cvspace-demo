CV Builder
===========

[![Build Status](https://travis-ci.org/Kinto/formbuilder.svg?branch=master)](https://travis-ci.org/Kinto/formbuilder)

If you want to try it out, have a look [at the demo
page](https://kinto.github.io/formbuilder/)

Or deploy it on Scalingo in a single click on this button:

[![Deploy to Scalingo](https://cdn.scalingo.com/deploy/button.svg)](https://my.scalingo.com/deploy)

Scalingo offer a 1 month free trial then 7.20€ / month.

# Installation

To run the formbuilder locally, you can issue the following commands:

```
$ git clone https://github.com/Kinto/formbuilder.git  
$ cd formbuilder  
$ npm install
$ npm run start  
```

You also need to have a [Kinto](https://kinto.readthedocs.io) server **greater
than 4.3.1** in order to store your data. If you don't already have one, follow [the
installation instructions](http://kinto.readthedocs.io/en/stable/tutorials/install.html)!

# Configuration

It's possible to configure a few things, using environment variables:

- `PROJECT_NAME` is the name of the project. Defaults to "cvbuilder".
- `SERVER_URL` is the URL of the kinto server. It's default value depends on
  the environment that's being used (development, production, etc.)

# How can I get my data back?

All the data generated by the formbuilder is stored in a
[Kinto](https://kinto.readthedocs.io) instance. As such, you can request the
data stored in there.

When you generate a form, you're actually generating two *tokens*:
- the *adminToken*, that you need to keep secret, giving you access to all the
  submitted data;
- the *userToken*, that's used by users to find back the proper form.

One interesting property of the *userToken* is that it is actually half of the
admin token !

With that in mind, let's say we've generated a form with an *adminToken* of `152e3b0af1e14cb186894980ecac95de`. The *userToken* is then `152e3b0af1e14cb1`.

So if we want to have access to the data on the server, using curl, we need to authenticate as the admin (using [BasicAuth](https://en.wikipedia.org/wiki/Basic_access_authentication) with `form:{adminToken}`):

```
$ SERVER_URL="http://localhost:8888/v1"
$ ADMIN_TOKEN="152e3b0af1e14cb186894980ecac95de"
$ FORM_ID="152e3b0af1e14cb1"
$ curl $SERVER_URL/buckets/formbuilder/collections/$FORM_ID/records \
   -u form:$ADMIN_TOKEN | python -m json.tool
{
    "data": [
        {
            "how_are_you_feeling_today": "I don't know",
            "id": "7785a0bb-cf75-4da4-a757-faefb30e47ae",
            "last_modified": 1464788211487,
            "name": "Clark Kent"
        },
        {
            "how_are_you_feeling_today": "Quite bad",
            "id": "23b00a31-6acc-4ad2-894c-e208fb9d38bc",
            "last_modified": 1464788201181,
            "name": "Garfield"
        },
        {
            "how_are_you_feeling_today": "Happy",
            "id": "aedfb695-b22c-433d-a104-60a0cee8cb55",
            "last_modified": 1464788192427,
            "name": "Lucky Luke"
        }
    ]
}
```

# Architecture of the project

The formbuilder is based on top of React and the [react-jsonschema-form (rjsf)](https://github.com/mozilla-services/react-jsonschema-form)
library.

It is also using [redux](https://github.com/reactjs/react-redux) to handle
the state and dispatch actions. If you're not familiar with it, don't worry,
here is an explanation of how it works.

## A quick theory tour

With react and redux, components are rendering themselves depending of some *state*. The state is passed to a component and becomes a set of `props`.

States aren't all stored at the same place and are grouped in so called *stores* (which are containers for the different states).

Now, if one needs to update the props of a component, it will be done via an *action*.

An *action* will eventually trigger some changes to the state, by the way of *reducers*: they are simple functions which take a original state, an action and return a new version of the state.

Then, the state will be given to the components which will re-render with the new values. As all components don't want to have access to all stores, the mapping is defined in a "container".

Yeah, I know, it's a bunch of concepts to understand. Here is a short recap:

state
: The state of the application. It's stored in different stores.

actions
: triggered from the components, they usually do something and then are relayed
  to reducers in order to update the state.

reducers
: Make the state of a store evolve depending a specified action

containers
: Containers define which stores and actions should be available from a react
  component.

## How are components organised ?

At the very root, there is an "App" component, which either:
- renders a "mainComponent"
- renders a sidebar (customisable), the notifications and a content (customisable)

It is useful to have such a root component for the styling of the application:
header is always in place and the menu always remains at the same location as
well, if there is a need for such a menu.

There are multiple "screens" that the user navigates into, which are summarized
here.

This might not be completely up to date, but is here so you can grasp easily how things are put together in the project.

### Welcome screen

There is a `Welcome` component which is rendered as a mainComponent.

```
<App>
  <Welcome />
</App>
```

### Form edition

This is the "builder" part of the formbuilder.

On the left side, you have a FieldList component and on the
right side you have the components. It's possible to
dragndrop from the fieldlist to the form.

Each time a field is added to the form, the `insertfield` action is called.

```
<App>
  <Header />
  <NotificationList />
  <FieldList>
    <Draggable />
    <Draggable />
    ...
  </FieldList />
  <Form>
    <EditableField>
      <FieldPropertiesEditor /> or <SchemaField />
    </EditableField>
    <Droppable />
  <Form />
</App>
```

The form is actually rendered by the rjsf library using a custom SchemaField, defined in `components/builder/EditableField.js`.

### Form created confirmation

Once the form is created, the user is shown the FormCreated view, with a checkbox on the left side. This has two links to the AdminView and the UserForm views.

```
<App>
  <Header />
  <NotificationContainer />
  <Check />
  <FormCreated />
</App>
```

### Form administration

The form admin is where the answers to the forms can be viewed. It should be viewable only by the people with the administration
link.

```
<App>
  <AdminView>
    <CSVDownloader />
  </AdminView>
</App>
```

### Form user

The UserForm is what the people that will fill the form will see.

```
<App>
  <UserForm>
</App>
```
