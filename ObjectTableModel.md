# Index #

---

1. Motivation

2. Let’s Code

2.1 Basic

2.1.1 Introduction

2.1.2 Custom formatters.

2.1.3 Methods from the List interface.

2.1.4 Updating and getting values from the model.

2.2 Advanced

2.2.1 FieldResolver

2.2.2 FieldHandler and MethodHandler

---


## 1. Motivation ##
"DON'T use DefaultTableModel", it's common to me to meet people who got problems using the DefaultTableModel implementation and my tip is it, don't use then. But implement one who make all the work easy for us, are not so easy too. So I implemented one and come here to share with all.

My goal is to write a single TableModel, simple, extensible, legible and powerfull. And it's come possible with Reflection and Annotations.

With this model you:

•Add and get the current object at each row.

•Dont need to work with String arrays.

•Keep the objects updated at each update in the cell of the table.

•Configuration by Annotations who simplifies the code legibility.

•Methods from the List like: add, addAll, remove and indexOf.

•If you dont like annotations you can still use it.(See chapter 2.2.1),

## 2. Let's Code ##
### 2.1 Basic ###
#### 2.1.1 Introduction ####
First: Download the objecttablemodel.zip archive who contains the source code of the project.

The interesting classes of this project are.

ObjectTableModel who are the table model implemented.

FieldResolver the background work are done here accessing the field for the Table cols.

@Resolvable the annotation who mark the fields its default values to the table. Like formatter if needed, Column name and the FieldAccessHandler who really access the field.

The implemented FieldAccessHandlers are FieldHandler(default) who use directly the Field in the class, and MethodHandler who use the get(or is)/set methods in the class.

The AnnotationResolver class just has common Methods for creating FieldResolvers.

And it's the only code we need to create a JTable of a class.

First: The class, here as example i'll use Person.

```
import com.towel.el.annotation.Resolvable;

public class Person {
    @Resolvable(colName = "Name")
    private String name;
    @Resolvable(colName = "Age", formatter = IntFormatter.class)
    private int age;
    private Person parent;

    public Person(String name, int age) {
        this(name, age, null);
    }

    public Person(String name, int age, Person parent) {
        this.name = name;
        this.age = age;
        this.parent = parent;
    }
    //Getters and setters ommited 
} 
```

And the code we need to create a table are just that.

```
import java.awt.Dimension;
import java.util.ArrayList;
import java.util.List;

import javax.swing.JFrame;
import javax.swing.JScrollPane;
import javax.swing.JTable;

import com.towel.el.annotation.AnnotationResolver;
import com.towel.swing.table.ObjectTableModel;

import test.Person;

public class ObjectTableModelDemo {
    public void show() {
	//Here we create the resolver for annotated classes.
        AnnotationResolver resolver = new AnnotationResolver(Person.class);
	//We use the resolver as parameter to the ObjectTableModel
	//and the String represent the cols.
        ObjectTableModel<Person> tableModel = new ObjectTableModel<Person>(
                resolver, "name,age");
	//Here we use the list to be the data of the table.
        tableModel.setData(getData());

        JTable table = new JTable(tableModel);
        JFrame frame = new JFrame("ObjectTableModel");
        JScrollPane pane = new JScrollPane();
        pane.setViewportView(table);
        pane.setPreferredSize(new Dimension(400,200));
        frame.add(pane);
        frame.pack();
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    }
    //Just for create a default List to show.
    private List<Person> getData() {
        List<Person> list = new ArrayList<Person>();
        list.add(new Person("Marky", 17, new Person("Marcos", 40)));
        list.add(new Person("Jhonny", 21));
        list.add(new Person("Douglas", 50, new Person("Adams", 20)));
        return list;
    }

    public static void main(String[] args) {
        new ObjectTableModelDemo().show();
    }
} 
```

The Second parameter of the ObjectTableModel class can be more powerfull than this.

You are not limited to the attributes of this class. You can use the attributes of the fields in that.

```
AnnotationResolver resolver = new AnnotationResolver(Person.class);
        ObjectTableModel<Person> tableModel = new ObjectTableModel<Person>(
                resolver, "name,age,parent.name,parent.age"); 
```

In this case, using "parent.name" you see the name of the parent of this Person.


You can especify the column name too. Just put a colon (:) after the field name and write the column name.

```
 AnnotationResolver resolver = new AnnotationResolver(Person.class);
  ObjectTableModel<Person> tableModel = new ObjectTableModel<Person>(
   resolver, "name:Person Name,age:Person Age,parent.name:Parent Name,parent.age:Parent Age"); 
```

#### 2.1.2 Custom Formatters ####
In most cases only String is enough to suply the correct visualization.

But if we need to place a Calendar field in our table?
```
java.util.GregorianCalendar[time=-367016400000,areFieldsSet=true,areAllFieldsSet=
true,lenient=true,zone=sun.util.calendar.ZoneInfo[id="America/Sao_Paulo",offset=
-10800000,dstSavings=3600000,useDaylight=true,transitions=129,lastRule=java.util.
SimpleTimeZone[id=America/Sao_Paulo,offset=-10800000,dstSavings=3600000,useDaylight=
true,startYear=0,startMode=3,startMonth=9,startDay=15,startDayOfWeek=1,startTime=0,
startTimeMode=0,endMode=3,endMonth=1,endDay=15,endDayOfWeek=1,endTime=0,endTimeMode=
0]],firstDayOfWeek=2,minimalDaysInFirstWeek=1,ERA=1,YEAR=1958,MONTH=4,WEEK_OF_YEAR=
20,WEEK_OF_MONTH=3,DAY_OF_MONTH=16,DAY_OF_YEAR=136,DAY_OF_WEEK=6,DAY_OF_WEEK_IN_MONTH
=3,AM_PM=0,HOUR=0,HOUR_OF_DAY=0,MINUTE=0,SECOND=0,MILLISECOND=0,ZONE_OFFSET=
-10800000,DST_OFFSET=0]
```
It is not suitable for view.

For this reason we create a new instace of the com.towel.bean.Formatter.



It's methods signatures are.

```
package com.towel.bean;

/**
 *@author Marcos Vasconcelos
 */
public interface Formatter {
	/**
	 * Convert a object to String.
	 */
	public abstract Object format(Object obj);

	/**
	 * Convert the String to the Object.
	 */
	public abstract Object parse(Object s);

	/**
	 * Naming proposes only
	 */
	public abstract String getName();
}
```

We can set the formatter in the @Resolvable annotation and there is my implementation for calendar.


---

Important, when implementing a Formatter Formatter, make sure to asign the method format with the return wich you wish to show in the table, in this case, format to String.

---


```
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.GregorianCalendar;

import com.towel.bean.Formatter;

public class CalendarFormatter implements Formatter {
    private final static SimpleDateFormat formatter = new SimpleDateFormat(
            "dd/MM/yyyy");

    @Override
    public String format(Object obj) {
        Calendar cal = (Calendar) obj;
        return formatter.format(cal.getTime());
    }

    @Override
    public String getName() {
        return "calendar";
    }

    @Override
    public Object parse(String s) {
        Calendar cal = new GregorianCalendar();
        try {
            cal.setTime(formatter.parse(s));
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return cal;
    }

}
```

Returning to the class Person we can now create another field and place in our Table.

```
@Resolvable(formatter = CalendarFormatter.class)
private Calendar birth;
```

And for our table we can use String: "name,age,birth"

And the third column will have a value like "26/06/1991".

More suitable for view instead the standard Calendar.toString().

#### 2.1.3 Methods from the List interface. ####
I added methods from the List interface in the model and it's become simple to add and remove objects as we do on lists.

And here a example with thoses methods.

```
	ObjectTableModel<Person> model = new ObjectTableModel<Person>(
		new AnnotationResolver(Person.class).resolve("name,age"));
	Person person1 = new Person("Marky", 17);
	Person person2 = new Person("MarkyAmeba", 18);
	model.add(person1);
	model.add(person2);

	List<Person> list = new ArrayList<Person>();
	list.add(new Person("Marcos", 40));
	list.add(new Person("Rita", 40));

	model.addAll(list);

	int index = model.indexOf(person2);// Should return 2
	model.remove(index);

	model.remove(person1);

	model.clean(); 
```

#### 2.1.4 Updating and getting objects from the model ####
Of course. A table is not only for show data. In the model theres a method called setEditDefault  and the method isEditable(int x, int y) return this value.(It means if it's set to true, all table is editable. And false all table is not editable).

If it's set to true you can edit the cells. After the focus lost the table call the setValueAt in the Model and it's set in the properly object.

The value are passed as String and the FieldResolver use it's Formatter instance to convert the value to set in the object. It means you are not limited to work with Strings, but any Object. Implementing a correct Formatter it's come possible.

And here a example how it's work.

First. Our model and the Formatter.
```
import com.towel.el.annotation.Resolvable;
import com.towel.el.handler.MethodHandler;

public class Person {
	@Resolvable(colName = "Name")
	private String name;
	@Resolvable(colName = "Age", formatter = IntFormatter.class)
	private int age;
	private Person parent;

	public Person(String name, int age, Person parent) {
		this.name = name;
		this.age = age;
		this.parent = parent;
	}

	public Person(String name, int age) {
		this.name = name;
		this.age = age;
	}
public static class IntFormatter implements Formatter {
	@Override
	public String format(Object obj) {
		return Integer.toString((Integer) obj);
	}

	@Override
	public String getName() {
		return "int";
	}

	@Override
	public Object parse(String s) {
		return Integer.parseInt(s);
	}
}
}
```
Here. The example.
```
import java.awt.Dimension;
import java.util.ArrayList;
import java.util.List;

import javax.swing.JFrame;
import javax.swing.JScrollPane;
import javax.swing.JTable;

import com.towel.el.annotation.AnnotationResolver;
import com.towel.swing.table.ObjectTableModel;

public class AnnotationResolverTest {
	public void testAnnotationResolverInit() {
		AnnotationResolver resolver = new AnnotationResolver(Person.class);
		ObjectTableModel<Person> tableModel = new ObjectTableModel<Person>(
				resolver,
				"name,age,parent.name:Parent,parent.age:Parent age");
		tableModel.setData(getData());
		tableModel.setEditableDefault(true);

		JTable table = new JTable(tableModel);
		JFrame frame = new JFrame("ObjectTableModel");
		JScrollPane pane = new JScrollPane();
		pane.setViewportView(table);
		pane.setPreferredSize(new Dimension(400, 200));
		frame.add(pane);
		frame.pack();
		frame.setLocationRelativeTo(null);
		frame.setVisible(true);
		frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
	}

	private List<Person> getData() {
		List<Person> list = new ArrayList<Person>();
		list.add(new Person("Marky", 17, new Person("Marcos", 40)));
		list.add(new Person("Jhonny", 21, new Person("",0)));
		list.add(new Person("Douglas", 50, new Person("Adams",20)));
		return list;
	}

	public static void main(String[] args) {
		new AnnotationResolverTest().testAnnotationResolverInit();
	}
}
```

Any change in the cell will update the object.

##### Getting the object of the row. #####

The worse part to work with JTables are getting it's values. Almost all time we need to get the values with getValutAt and set to the properly value in the object. But the aims of this project are make it come to pass.

The method getValue(int row) of the ObjectTableModel returns the Object of the especified row.

The ObjectTableModel are typed and the method getValue returns a object of the type, avoiding class casting.

The folowing code returns the Person at the second row.
```
               AnnotationResolver resolver = new AnnotationResolver(Person.class);
		ObjectTableModel<Person> tableModel = new ObjectTableModel<Person>(
				resolver,
				"name,age,parent.name:Parent,parent.age:Parent age");
		tableModel.setData(getData());
		tableModel.setEditableDefault(true);

		Person person = tableModel.getValue(2);//The row
		System.out.println(person.getName()); 
```
### 2.2 Advanced ###
All until here is enough to use the model. But is still possible extend the functions of the table to serve any purpose.
#### 2.2.1 FieldResolver ####
All the background of this project are in this class. The @Resolvable and the AnnotationResolver are only for create FieldResolver instances for the ObjectTableModel.

But you can still use it instead the annotations.

The following code.
```
		FieldResolver nameResolver = new FieldResolver(Person.class, "name");
		FieldResolver ageResolver = new FieldResolver(Person.class, "age");
		ageResolver.setFormatter(new IntFormatter());
		FieldResolver parentNameResolver = new FieldResolver(Person.class,
				"paren.name", "Parent");
		FieldResolver parentAgeResolver = new FieldResolver(Person.class,
				"parent.age", "Parent age");
		FieldResolver birthResolver = new FieldResolver(Person.class, "birth",
				"Birth day");
		birthResolver.setFormatter(new CalendarFormatter());
		ObjectTableModel<Person> model = new ObjectTableModel<Person>(
				new FieldResolver[] { nameResolver, ageResolver,
				parentNameResolver, parentAgeResolver, birthResolver });
```
Are equivalent for this code(assuming the Person class is properly annotated with @Resolvable annotations).
```
		AnnotationResolver resolver = new AnnotationResolver(Person.class);
		ObjectTableModel<Person> tableModel = new ObjectTableModel<Person>(
				resolver,
		"name,age,parent.name:Parent,parent.age:Parent age,birth: Birth day");
		tableModel.setData(getData());
```
##### Field Resolver Factory #####
The FieldResolverFactory are only for code legibility to make it easy to create new FieldResolvers. It's constructor need a Class<?> object who represent the class who we are creating field resolvers(The same as we pass in the FieldResolver constructor)

> The  main methods are:

createResolver(String fieldName).

createResolver(String fieldName, String colName).

createResolver(String fieldName, Formatter formatter).

createResolver(String fieldName, String colName, Formatter formatter).

The first example using FieldResolver using the factory should be.
```
		FieldResolverFactory fac = new FieldResolverFactory(Person.class);
		FieldResolver nameRslvr = fac.createResolver("name");
		FieldResolver ageRslvr = fac.createResolver("age", new IntFormatter());
		FieldResolver parentNameRslvr = fac.createResolver("paren.name",
				"Parent");
		FieldResolver parentAgeRslvr = fac.createResolver("parent.age",
				"Parent age", new IntFormatter());
		FieldResolver birthRslvr = fac.createResolver("birth", "Birth day",
				new CalendarFormatter());
		ObjectTableModel<Person> model = new ObjectTableModel<Person>(
				new FieldResolver[] { nameRslvr, ageRslvr, parentNameRslvr,
						parentAgeRslvr, birthRslvr });
```

#### 2.2.2 FieldHandler and MethodHandler. ####
Until here we are using the default FieldAccessHandler who are the FieldHandler.

Using this, we dont need any getter/setters for our attributes in the class. All are accessed directly by Reflection.

Using the MethodHandler the class search the getter or is/setter methods in the given class.

A simple example.
```
import java.util.ArrayList;
import java.util.Calendar;
import java.util.LinkedList;
import java.util.List;

import com.towel.el.annotation.Resolvable;
import com.towel.el.handler.MethodHandler;

public class Person {
	@Resolvable(colName = "Name", accessMethod = MethodHandler.class)
	private String name;
	@Resolvable(colName = "Age", formatter = IntFormatter.class)
	private int age;

	public Person(String name, int age) {
		this.name = name;
		this.age = age;
	}

	public String getName() {
		return "The name is: " + name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public int getAge() {
		return 150;
	}

	public void setAge(int age) {
		this.age = age;
	}
}
```

And the init of the TableModel.

```
AnnotationResolver resolver = new AnnotationResolver(Person.class);
		ObjectTableModel<Person> tableModel = new ObjectTableModel<Person>(
				resolver,
				"name,age");
		tableModel.setData(getData());
```

Running this we note in all the name columns a String starting "The name is: " cause it's in the getName method and we are using MethodHandler for this field. But for the getAge who always return 150 we can note the actual age attribute set cause it still use the FieldHandler.