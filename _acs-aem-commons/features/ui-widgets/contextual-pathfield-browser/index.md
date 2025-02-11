---
layout: acs-aem-commons_feature
title: Contextual Pathfield Browser
description: Dynamically limit selectable paths!
date: 2017-02-11
tags: aem-65 aem-cs
initial-release: 2.10.0
---

## Contextual Path Browser
The `contextualpathbrowser` Granite UI widget functions just like a standard `pathbrowser` widget other than it detects
the root page of the current site on a multi-site platform and limits the author to selecting content within that site.

Take the following content structure:

    /content
        /siteA
            /pageA1
            /pageA2
        /siteB
            /pageB1
            /pageB2

For a component that requires a standard `pathbrowser` field to be used on both of these sites, the `pathbrowser` would
need to be configured with a root of `/` or `/content`, leaving the author to correctly navigate into siteA when working
on siteA, or into siteB when working on siteB. As the number of sites increase, so does the chance for user error. It
also makes for a less clean authoring experience.

The `contextualpathbrowser` field instead leverages the `ACS AEM Commons - Page Root Provider Configuration` configured via OSGi to
determine the base page of the current site and dynamically limit the content to that particular site. As such, an
author opening up a dialog with `contextualpathbrowser` from any page on siteA will see and be able to select pages
only from siteA, and an author on siteB will see and be able to select pages only from siteB.

To use `contextualpathbrowser`, you must first configure the `ACS AEM Commons - Page Root Provider Configuration` in OSGi with the appropriate path(s)
or regex(es) to determine the root page(s) of your site(s).  Then, to leverage `contextualpathbrowser` in a dialog,
simply replace

    sling:resourceType="granite/ui/components/foundation/form/pathbrowser"

with

    sling:resourceType="acs-commons/touchui-widgets/contextualpathbrowser"
    
Other than `rootPath`, which is determined dynamically by `contextualpathbrowser`, all configurations are the same as
with the standard `pathbrowser` widget.

## Vanity Path Uniqueness Check (Since 1.5.0)

The `uniqueVanityPath` vtype ensures that a `sling:vanityPath` value is unique across a particular installation of AEM. To enable this, create a copy of `/libs/foundation/components/page/tab_basic` specific to your page component(s) and then reference this copy from your page component(s) dialog. Edit the node at `items/vanity/items/vanityPath` so that only these properties are set:

    xtype: cqinclude
    path:  /apps/acs-commons/components/utilities/unique-vanity-path/vanityPath.infinity.json

This creates the field with the appropriate field validator.

## Multi Field Panel (Since 1.5.0)

The `multifieldpanel` widget is a widget which enables the management of complex items in a traditional `multifield`. These complex items are composed of multiple widgets. For example, a field containing terms and definitions would be configured using this node structure:

     <definitions
         jcr:primaryType="cq:Widget"
         fieldLabel="Definitions"
         name="./definitions"
         orderable="{Boolean}true"
         xtype="multifield">
         <fieldConfig
             jcr:primaryType="cq:Widget"
             xtype="multifieldpanel">
             <items jcr:primaryType="cq:WidgetCollection">
                 <term
                     jcr:primaryType="cq:Widget"
                     fieldLabel="Term"
                     key="term"
                     width="250"
                     xtype="textfield"/>
                 <definition
                     jcr:primaryType="cq:Widget"
                     fieldLabel="Definition"
                     key="definition"
                     width="250"
                     xtype="textarea"/>
             </items>
         </fieldConfig>
     </definitions>

> A similar widget has circulated in the past under the name `multifieldmultifield`.

This widget persists its data into a single JSON-encoded string per item where each data element's key in the JSON object is defined in the `key` configuration property. For example, this dialog entry:

![MultiFieldPanel in Dialog]({{ site.data.acs-aem-commons.baseurl }}/images/widgets/multifieldpanel-dialog.png)

Would be stored as:

    {"term":"term","definition":"a word or phrase used to describe a thing"}

To ease working with these JSON objects, a JSP function exists:

    <%@ taglib prefix="widgets" uri="http://www.adobe.com/consulting/acs-aem-commons/widgets" %>
    <c:set var="definitions" value="${widgets:getMultiFieldPanelValues(resource, 'definitions')}"/>

At this point, `definitions` contains a `List<Map<String, String>>` which can then be iterated through using `<c:forEach>` (or any other way you might iterate over a collection):

    <c:forEach items="${definitions}" var="definition">
        <dt>${xss:encodeForHTML(xssAPI, definition['term'])}</dt>
        <dd>${xss:encodeForHTML(xssAPI, definition['definition'])}</dt>
    </c:forEach>
        
To work with these JSON objects in sightly, a WCMUse class (since version 2.9.0/3.6.0) exists:

       <sly data-sly-use.multiField="${ 'com.adobe.acs.commons.widgets.MultiFieldPanelWCMUse' @ name='definitions' }">
       
At this point, `multiField` contains an instance of the class and the property `values` can be iterated through using `data-sly-list`:
       
       <dl data-sly-list.definition="${ multiField.values }">
            <dt>${ definition.term }</dt>
            <dd>${ definition.definition }</dd>
       </dl>
 
It is possible to retrieve values from diferent resources setting the `location` parameter in the WCMUse initialization. If none is set, the current resource is used. 

## Drag-and-Drop Path Field (Since 1.1.0)

The `ddpathfield` widget is a drop-in replacement for the standard [`pathfield`](http://dev.day.com/docs/en/cq/current/widgets-api/index.html?class=CQ.form.PathField) widget. It supports all the same capabilities of the `pathfield` and adds the ability to drag-and-drop pages and assets from the Content Finder. It works both by itself and within a `multifield`.

By default, the will accept either pages or assets. This behavior can be restricted by setting the `ddGroups` configuration property to one or more of the `DD_GROUP` constants from [`CQ.wcm.EditBase`](http://dev.day.com/docs/en/cq/current/widgets-api/index.html?class=CQ.wcm.EditBase):

* `media`
* `page`
* `editcomponent`
* `default`
* `paragraph`
* `product`

## MultiField Minimum and Maximum Validator (Since 1.0.0)

When installed, ACS AEM Commons seamlessly adds support to the existing [`CQ.form.MultiField`](http://dev.day.com/docs/en/cq/current/widgets-api/index.html?class=CQ.form.MultiField) widget to provide a minimum and/or maximum number of items. These are defined in configuration properties named `minItems` and `maxItems`.

## Additional Search Predicates (Since 1.0.0)

These additional search predicate widgets are available for use in the SiteAdmin and DAM Admin Search Panels:

* `authorizablepredicate` - Select a user or group, e.g. to search for assets who were last modified by a particular user.
* `selectionpredicate` - Select a value from a drop down list.
* `pathpropertypredicate` - Use a path field to select a path and then search for property values matching that path.

### Configuration Options

#### Authorizable Predicate

Configuration Options:

* `propertyName` - Name of property for search. Defaults to `jcr:content/jcr:lastModifiedBy`
* `predicateName` - QueryBuilder predicate name. Defaults to `property`
* `filter` - Authorizable type filter. Can be `groups` or `users` (defaults to `users`)

#### Selection Predicate

Configuration Options:

* `propertyName` - Name of property for search.
* `predicateName` - QueryBuilder predicate name. Defaults to `property`
* `options` - See `options` on [`CQ.form.Selection`](http://dev.day.com/docs/en/cq/current/widgets-api/?class=CQ.form.Selection)

#### Path Property Predicate

Configuration Options:

* `propertyName` - Name of property for search.
* `predicateName` - QueryBuilder predicate name. Defaults to `property`.
* `rootPath` - Root path for path selection. Defaults to `/content`.
* `pathFieldPredicateName` - See `predicate` on [`CQ.form.PathField`](http://dev.day.com/docs/en/cq/current/widgets-api/?class=CQ.form.PathField)
