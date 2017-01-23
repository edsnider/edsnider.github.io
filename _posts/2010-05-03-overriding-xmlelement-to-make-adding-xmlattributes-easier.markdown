---
layout: post
title:  "Overriding XmlElement to Make Adding XmlAttributes Easier"
date:   2010-05-03 00:00:00 -0500
tags: C# xml xmlelement
---

When programmatically building Xml documents adding elements and their attributes can become rather repetitive and tedious. Once you create the element object you have to create separate `XmlAttribute` instances for each attribute of that element tag and add them to the element’s attribute collection. In many cases you will have to add the same attribute or common set of attributes to different elements throughout the document. This can be a lot easier and intuitive by overriding the `XmlElement` class and making the element’s properties that would be `XmlAttribute` instances actual instance fields (public properties) of the `XmlElement` object itself. So, the client end of your overridden `XmlElement` object will simply set the element’s attribute values just as it would the `InnerText` property, instead of having create `XmlAttribute` instances. The implementation within the overridden `XmlElement` will handle the construction of the `XmlAttribute` instances as well as adding them to its attributes collection.

By overriding `XmlElement` and abstracting away its attribute collection the client code can instantiate and construct `XmlElement`s that are specific to the enterprise, much like it would normal business logic entities… and save a few lines of code along the way. See a quick comparison between the typical way of adding attributes to an Xml node and the easier way after overriding `XmlElement`.

The typical way – setting up an `XmlAttribute` object for each attribute of the `XmlElement`:

{% highlight csharp %}
XmlElement employeeElement;
XmlAttribute employeeAttribute;
employeeElement = xmlDoc.CreateElement("Employee");
employeeAttribute = xml.CreateAttribute("ci:emailaddress");
employeeAttribute.Value = "sample.emp@company.com";
employeeElement.Attributes.Append(employeeAttribute);
employeeAttribute = xml.CreateAttribute("ci:phone");
employeeAttribute.Value = "123-456-7890";
employeeElement.Attributes.Append(employeeAttribute);
employeeAttribute = xml.CreateAttribute("ci:fax");
employeeAttribute.Value = "123-456-7899";
employeeElement.Attributes.Append(employeeAttribute);
{% endhighlight %}

The easier way – adding each attribute of the `XmlElement` as instance data:

{% highlight csharp %}
ContactXmlElement employeeElement;
employeeElement = (ContactXmlElement)xmlDoc.CreateElement("Employee");
employeeElement.EmailAddress = "sample.emp@company.com";
employeeElement.Phone = "123-456-7890";
employeeElement.Fax = "123-456-7899";
{% endhighlight %}

In order to override `XmlElement` you will also have to override the `CreateElement` method of the `XmlDocument` class so you can construct objects based on your overridden implementation of `XmlElement` from the document object like you would normally do when creating new `XmlElements`.

#### Step 1: 

Create a class that overrides `XmlDocument` and include an overriding implementation of the `CreateElement` method. Also, it is a good idea to add a public field for the attributes’ namespace if they are called from a separate namespace than the rest of the document (i.e. `ci:emailAddress`).

{% highlight csharp %}
using System;
using System.Xml; 
 
namespace EdSnider.Samples
{
    public class CompanyXmlDocument : XmlDocument
    {
        public string CINamespaceURI { get; set; } 
 
        public override XmlElement CreateElement(string prefix, string localName, string namespaceURI)
        {
            if (localName == "Customer" || localName == "Employee")
            {
                return new ContactXmlElement(prefix, localName, namespaceURI, this, this.CINamespaceURI);
            }
            else
            {
                return base.CreateElement(prefix, localName, namespaceURI);
            }
        }
    }
}
{% endhighlight %}

#### Step 2: 

Create a class to override `XmlElement`. In this implementation of `XmlElement` include public instance fields for each of the element’s properties that would you normally be creating `XmlAttribute`s for. In the example below I am overriding `XmlElement` to make an `ContactXmlElement` class. In this `ContactXmlElement` class I am including public instance fields for Phone, Email, and Fax.

{% highlight csharp %}
using System;
using System.Xml; 
 
namespace EdSnider.Samples
{
    public class ContactXmlElement : XmlElement
    {
        private string _phone;
        private string _emailAddress;
        private string _fax;
        private string _ciNamespaceUri; 
 
        public string Phone
        {
            get { return this._phone; }
            set { this._phone = value; base.SetAttribute("phone", this._ciNamespaceUri, value); }
        } 
 
        public string EmailAddress
        {
            get { return this._emailAddress; }
            set
            {
                this._emailAddress = value;
                base.SetAttribute("emailAddress", this._ciNamespaceUri, value);
            }
        } 
 
        public string Fax
        {
            get { return this._fax; }
            set
            {
                this._fax = value;
                base.SetAttribute("fax", this._ciNamespaceUri, value);
            }
        } 
 
        internal ContactXmlElement(string prefix, string localName, string namespaceURI, XmlDocument doc, string CINamespaceURI) : base(prefix, localName, namespaceURI, doc)
        {
            this._ciNamespaceUri = CINamespaceURI;
        }
    }
}
{% endhighlight %}

#### Step 3: 

Use the overridden versions of `XmlDocument` and `XmlElement` to build a XML document within the client code:

{% highlight csharp %}
CompanyXmlDocument xmlDoc = new CompanyXmlDocument();
xmlDoc.CINamespaceURI = "uri:edsnider:ci";
xmlDoc.LoadXml(""); // make sure you include all the appropriate namespaces within the tag – I omitted them in this sample for clarity
 
ContactXmlElement empElement = (ContactXmlElement)xmlDoc.CreateElement("Employee");
empElement.Phone = "123-456-7890";
empElement.EmailAddress = "sample.emp@company.com";
empElement.Fax = "123-456-7899";
empElement.InnerText = "Sample Employee"; 
 
xmlDoc.DocumentElement.AppendChild(empElement);
{% endhighlight %}

Notice that you can now set the value of the XML element attributes via the element’s object fields just like you would set the value of one of the standard `XmlElement` properties like `InnerText`.

The code in the previous snippet will create the following XML:

{% highlight xml %}
<company>
    <employee ci:emailaddress="sample.emp@company.com" ci:phone="123-456-7890" ci:fax="123-456-7899">
        Sample Employee
    </employee>
</company>
{% endhighlight %}