13/12/2010: Changelog in version 1.0

The version 1.0 of this project, is the version 2.8 of the Mark Utils project, plus the new features.

The TableFilter, made by ViniGodoy was added to this project under the com.towel.swing.table package.
And the JImagePanel, made by ViniGodoy was added too, under com.towel.swing.img package.

New article about it soon in Wiki.

All packages renamed, from mark.utils.`<packages>` to com.towel.`<packages>`.

Until it's done, some articles can still refers to the old project, I'll be changing it over time.



---

From here, all changes are from the old project.

---

11/13/2010: Changes in version 2.8

Class Expression totally remade, now with some more new features.

New feature, AggregateFunctions in Collections package.

All changes in their respective Wiki pages.

10/23/10: Changes in version 2.7.2

Fixed the problem about the formatter, sorry about it.
Now it can be used again, but when writing a Formatter, make sure to make the format method return the same type you want to show at cells.

Eg.
public String format(Object obj)//If you want to format for String

Is possible to format to another class of objects and use a especial Renderer for that type.

A new method just for convenience, isEmpty on ObjectTableModel.

08/29/10: Changes in version 2.7

Now all primitive types are supported without custom formatters.

TableCellRenderers can be show at the Jtable cells.

More info in portuguese:
http://markyameba.wordpress.com/2010/08/29/renderers-customizados-objecttablemodel/

I'm working in the JavaDocs of the project. I hope upload it soon.

04/29/10: Changes in version 2.6

Packed the new ActionManager.
Portuguese article in: http://markyameba.wordpress.com/2010/04/29/actionmanager (Soon in wiki)

Deprecated the old ActionManager and Action classes.(The new one should use the normal ActionListener instead of Action)

Prototype of a General Manager to init all the managers of this project.


There's some new articles in the Wiki page.

03/03/10: Changes in version 2.5

Packed the new Binder class, now with annotations.
You can see a portuguese article about Binder in:
http://markyameba.wordpress.com/2010/03/03/binder-2-0-agora-com-annotations

Soon I post all the articles in the Wiki page.
Minor changes in some classes.


Using the power of reflection this project aims to make the way to work with models (such TableModel, ComboBoxModel, etc.) and binders easy.

Check out the ObjectTableModel in this project and forgot about these trouble DefaultTableModel.

Some articles about this project can be found in portuguese in:
http://markyameba.wordpress.com

For a english version of the ObjectTableModel article
http://code.google.com/p/markutils/wiki/ObjectTableModel

And there's a lot of usefull classes in this project too.