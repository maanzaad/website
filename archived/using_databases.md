---
title: "Using Databases"
lastmodified: '2011-07-27'
redirect_from:
  - /Using_Databases/
---

Using Databases
===============

Mono can interact with a variety of commercial and open source databases using ADO.NET. The project's current goal for data access is to be compatible with the APIs exposed in .NET

Mono supports the System.Data stack, an extensive set of providers as well as [LINQ to SQL](http://msdn.microsoft.com/en-us/library/bb386976.aspx) (largely through the DbLinq project).

This document does not aim to be a tutorial on ADO.NET, since there are many fine books and online resources that can help you, only a guide to using databases with Mono.

Database Providers
==================

The following database providers are available for Mono applications:

-   [MySQL]({{ site.github.url }}/MySQL "MySQL")
-   [ODBC]({{ site.github.url }}/ODBC "ODBC")
-   [SQLClient]({{ site.github.url }}/SQLClient "SQLClient")
-   [Oracle]({{ site.github.url }}/Oracle "Oracle")
-   [PostgreSQL]({{ site.github.url }}/PostgreSQL "PostgreSQL")
-   [Firebird and Interbase]({{ site.github.url }}/Firebird_Interbase "Firebird Interbase")
-   [IBM DB2]({{ site.github.url }}/IBM_DB2 "IBM DB2")
-   [OLE DB]({{ site.github.url }}/OLE_DB "OLE DB")
-   [SQLite]({{ site.github.url }}/SQLite "SQLite")
-   [Sybase]({{ site.github.url }}/Sybase "Sybase")
-   [TDS Providers]({{ site.github.url }}/TDS_Providers "TDS Providers")
-   [TDS Generic]({{ site.github.url }}/TDS_Generic "TDS Generic")
-   [MaxDB]({{ site.github.url }}/MaxDB "MaxDB")
-   [VistaDB](http://www.vistadb.com)

Tools
=====

The following Mono tools are useful when working with ADO.NET:

-   sqlsharp.exe (SQL\#), a command line query tool included with Mono to enter and execute SQL statements.
-   xsd.exe , XML Schema Definition tool
    -   **XDR to XSD**- used to generate an XML schema from an XDR (XML Data Reduced schema) file. XDR was used by Microsoft prior to XSD becoming a W3C recommendation, and it needs to be supported for legacy reasons.
    -   **XML to XSD**- used to generate an XML schema from an XML file.
    -   **XSD to DataSet**- used to generate DataSet classes from an XSD schema file. The DataSet classes created can then be used with XML data.
    -   **XSD to Classes**- used to generate classes from an XSD schema file. The classes created can be used with System.XML.Serialization.XMLSerializer to read and write XML code that follows the schema.
    -   **Classes to XSD**- used to generate an XML schema from type(s) in a assembly file. The XML schema created by the tool defines the XML format used by System.XML.Serialization.XMLSerializer.


