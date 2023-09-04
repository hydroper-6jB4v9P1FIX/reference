# XML

The language includes facilities for working with XML.

## Filter

Within a `xml_node.filter!` operation is visible a `q!` macro that accepts a single property (`my_custom_ns:tag_name`) or attribute (`@my_custom_ns:attribute_name`). `xml_node.filter!` returns `XmlList`.

When `q!` takes an attribute, it returns a `str` that is empty if not found. When `q!` takes a property (or tag name), it returns `Option<Xml>`.

Examples:

```ds
let result: XmlList = xml_node.filter!(q!(@id) == "2");
```

## Descendants

The `xml_node.descendants!` operation accepts a single property (`my_custom_ns:tag_name`) or attribute (`@my_custom_ns:attribute_name`). `xml_node.descendants!` returns `XmlList`.

Examples:

```ds
let desc: XmlList = xml_node.descendants!(tag_name);
```